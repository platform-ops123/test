pipeline {
    agent any

    environment {
        TOOLJET_BASE_URL = "${TOOLJET_BASE_URL}" // e.g., http://host.docker.internal:3000
        TOOLJET_ACCESS_TOKEN = credentials('TOOLJET_ACCESS_TOKEN')
    }

    parameters {
        choice(
            name: 'ACTION',
            choices: [
                'SETUP_GIT_CONFIG',
                'PUSH_TO_GIT',
                'CREATE_FROM_GIT',
                'SYNC_FROM_GIT',
                'DEPLOY'
            ],
            description: 'Select the Git sync action to perform'
        )

        string(name: 'APP_ID', defaultValue: '', description: 'App ID or slug (required for PUSH_TO_GIT, SYNC_FROM_GIT, DEPLOY)')
        string(name: 'VERSION_ID', defaultValue: '', description: 'Version ID or name (required for PUSH_TO_GIT)')
        string(name: 'COMMIT_MESSAGE', defaultValue: 'Automated commit from Jenkins', description: 'Commit message for PUSH_TO_GIT')

        string(name: 'ORG_ID', defaultValue: '', description: 'Organization ID (required for SETUP_GIT_CONFIG, CREATE_FROM_GIT)')
        string(name: 'GIT_URL', defaultValue: '', description: 'Git HTTPS URL (required for SETUP_GIT_CONFIG)')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch name (required for SETUP_GIT_CONFIG)')
        string(name: 'GITHUB_APP_ID', defaultValue: '', description: 'GitHub App ID (required for SETUP_GIT_CONFIG)')
        string(name: 'GITHUB_APP_INSTALL_ID', defaultValue: '', description: 'GitHub App Installation ID (required for SETUP_GIT_CONFIG)')
        text(name: 'GITHUB_APP_PRIVATE_KEY', defaultValue: '', description: 'GitHub App Private Key in PEM format (required for SETUP_GIT_CONFIG)')
    }

    stages {
        stage('Run ToolJet GitSync Action') {
            steps {
                script {
                    switch (params.ACTION) {
                        case 'SETUP_GIT_CONFIG':
                            setupGitConfig()
                            break
                        case 'PUSH_TO_GIT':
                            validate(params.APP_ID, 'APP_ID')
                            validate(params.VERSION_ID, 'VERSION_ID')
                            pushToGit(params.APP_ID, params.VERSION_ID, params.COMMIT_MESSAGE)
                            break
                        case 'CREATE_FROM_GIT':
                            validate(params.APP_ID, 'APP_ID')
                            validate(params.ORG_ID, 'ORG_ID')
                            createAppFromGit(params.APP_ID, params.ORG_ID)
                            break
                        case 'SYNC_FROM_GIT':
                            validate(params.APP_ID, 'APP_ID')
                            syncFromGit(params.APP_ID)
                            break
                        case 'DEPLOY':
                            validate(params.APP_ID, 'APP_ID')
                            deployApp(params.APP_ID)
                            break
                        default:
                            error "Invalid action: ${params.ACTION}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ ToolJet GitSync action '${params.ACTION}' completed successfully."
        }
        failure {
            echo "❌ ToolJet GitSync action '${params.ACTION}' failed. Check logs above."
        }
    }
}

def validate(value, fieldName) {
    if (!value?.trim()) {
        error "Missing required parameter: ${fieldName}"
    }
}

def setupGitConfig() {
    def payload = [
        organizationId: params.ORG_ID,
        gitUrl: params.GIT_URL,
        branchName: params.BRANCH_NAME,
        githubAppId: params.GITHUB_APP_ID,
        githubAppInstallationId: params.GITHUB_APP_INSTALL_ID,
        githubAppPrivateKey: params.GITHUB_APP_PRIVATE_KEY
    ]

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/organizations/git",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]],
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )

    if (response.status != 201) {
        error "Failed to set up Git config. Status: ${response.status}"
    }
}

def pushToGit(appId, versionId, message) {
    def payload = [commitMessage: message]

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps/${appId}/versions/${versionId}/git-sync/push",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]],
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )

    if (response.status != 201) {
        error "Failed to push to Git. Status: ${response.status}"
    }
}

def createAppFromGit(appId, orgId) {
    def payload = [appId: appId, organizationId: orgId]

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps?createMode=git",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]],
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )

    if (response.status != 201) {
        error "Failed to create app from Git. Status: ${response.status}"
    }
}

def syncFromGit(appId) {
    def response = httpRequest(
        httpMode: 'PUT',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps/${appId}?createMode=git",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]]
    )

    if (response.status != 200) {
        error "Failed to sync from Git. Status: ${response.status}"
    }
}

def deployApp(appId) {
    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps/${appId}/git-sync/release",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]]
    )

    if (response.status != 201) {
        error "Failed to deploy app. Status: ${response.status}"
    }
}
