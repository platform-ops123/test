pipeline {
    agent any

    environment {
        TOOLJET_BASE_URL = "${TOOLJET_BASE_URL}" // e.g., 'http://host.docker.internal:3000'
        TOOLJET_ACCESS_TOKEN = credentials('tooljet-access-token')
        ORGANIZATION_ID = "${ORGANIZATION_ID}" // ToolJet Organization ID or slug
        GIT_URL = "${GIT_URL}" // GitHub Repo URL
        BRANCH_NAME = "${BRANCH_NAME}" // e.g., 'main'
        GITHUB_APP_ID = "${GITHUB_APP_ID}"
        GITHUB_APP_INSTALLATION_ID = "${GITHUB_APP_INSTALLATION_ID}"
        GITHUB_APP_PRIVATE_KEY = credentials('github-app-private-key') // store as secret text
    }

    parameters {
        choice(
            name: 'ACTION',
            choices: ['SYNC_FROM_GIT', 'PUSH_TO_GIT', 'CREATE_FROM_GIT', 'DEPLOY', 'SETUP_GIT_CONFIG'],
            description: 'Select the Git sync action to perform'
        )
        string(name: 'APP_ID', defaultValue: '', description: 'ToolJet App ID or slug')
        string(name: 'VERSION_ID', defaultValue: '', description: 'Version ID or Name (for PUSH_TO_GIT)')
        string(name: 'COMMIT_MESSAGE', defaultValue: 'Automated commit from Jenkins', description: 'Git commit message')
    }

    stages {
        stage('Run Selected Action') {
            steps {
                script {
                    switch (params.ACTION) {
                        case 'SETUP_GIT_CONFIG':
                            setupGitConfiguration()
                            break
                        case 'SYNC_FROM_GIT':
                            validate(params.APP_ID, 'APP_ID')
                            syncFromGit(params.APP_ID)
                            break
                        case 'PUSH_TO_GIT':
                            validate(params.APP_ID, 'APP_ID')
                            validate(params.VERSION_ID, 'VERSION_ID')
                            pushToGit(params.APP_ID, params.VERSION_ID, params.COMMIT_MESSAGE)
                            break
                        case 'CREATE_FROM_GIT':
                            validate(params.APP_ID, 'APP_ID')
                            createAppFromGit(params.APP_ID)
                            break
                        case 'DEPLOY':
                            validate(params.APP_ID, 'APP_ID')
                            deployApp(params.APP_ID)
                            break
                        default:
                            error "❌ Invalid action: ${params.ACTION}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ ToolJet GitSync operation '${params.ACTION}' completed successfully."
        }
        failure {
            echo "❌ ToolJet GitSync operation '${params.ACTION}' failed. Check logs above for details."
        }
    }
}

// --- Utilities ---
def validate(value, field) {
    if (!value?.trim()) {
        error "❌ ${field} is required for '${params.ACTION}'"
    }
}

// --- Setup Git Configuration ---
def setupGitConfiguration() {
    echo "🔧 Setting up Git config..."

    def payload = [
        organizationId           : env.ORGANIZATION_ID,
        gitUrl                   : env.GIT_URL,
        branchName               : env.BRANCH_NAME,
        githubAppId              : env.GITHUB_APP_ID,
        githubAppInstallationId : env.GITHUB_APP_INSTALLATION_ID,
        githubAppPrivateKey     : env.GITHUB_APP_PRIVATE_KEY
    ]

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/organizations/git",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]],
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )

    if (response.status == 201) {
        echo "✅ Git configuration set successfully."
    } else {
        error "❌ Failed to set Git configuration. Status: ${response.status}"
    }
}

// --- Sync App from Git ---
def syncFromGit(appId) {
    echo "🔄 Syncing app '${appId}' from Git..."

    def response = httpRequest(
        httpMode: 'PUT',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps/${appId}?createMode=git",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]]
    )

    if (response.status == 200) {
        echo "✅ App synced from Git."
    } else {
        error "❌ Failed to sync app from Git. Status: ${response.status}"
    }
}

// --- Push Version to Git ---
def pushToGit(appId, versionId, commitMessage) {
    echo "📤 Pushing version '${versionId}' of app '${appId}' to Git..."

    def payload = [commitMessage: commitMessage]

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps/${appId}/versions/${versionId}/git-sync/push",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]],
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )

    if (response.status == 201) {
        echo "✅ App version pushed to Git."
    } else {
        error "❌ Failed to push to Git. Status: ${response.status}"
    }
}

// --- Create App from Git ---
def createAppFromGit(appId) {
    echo "🆕 Creating app '${appId}' from Git..."

    def payload = [
        appId         : appId,
        organizationId: env.ORGANIZATION_ID
    ]

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps?createMode=git",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]],
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )

    if (response.status == 201) {
        def responseJson = readJSON text: response.content
        echo "✅ App created. ID: ${responseJson.id}"
    } else {
        error "❌ Failed to create app from Git. Status: ${response.status}"
    }
}

// --- Deploy App ---
def deployApp(appId) {
    echo "🚀 Deploying app '${appId}'..."

    def response = httpRequest(
        httpMode: 'POST',
        url: "${env.TOOLJET_BASE_URL}/api/ext/apps/${appId}/git-sync/release",
        contentType: 'APPLICATION_JSON',
        customHeaders: [[name: 'Authorization', value: "Basic ${env.TOOLJET_ACCESS_TOKEN}"]]
    )

    if (response.status == 201) {
        echo "✅ App deployed to production."
    } else {
        error "❌ Failed to deploy app. Status: ${response.status}"
    }
}
