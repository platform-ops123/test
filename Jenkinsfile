pipeline {
  agent any

  environment {
    BASE_URL = credentials('TOOLJET_BASE_URL')
    TOKEN = credentials('TOOLJET_ACCESS_TOKEN')
    APP_ID = 'your-app-id-or-slug'
    VERSION_ID = 'your-version-name'
  }

  stages {
    stage('Push to GitHub') {
      steps {
        sh """
        curl -X POST "$BASE_URL/api/ext/apps/$APP_ID/versions/$VERSION_ID/git-sync/push" \
          -H "Authorization: Basic $TOKEN" \
          -H "Content-Type: application/json" \
          -d '{ "commitMessage": "CI: Jenkins Push" }'
        """
      }
    }

    stage('Pull from GitHub') {
      steps {
        sh """
        curl -X PUT "$BASE_URL/api/ext/apps/$APP_ID?createMode=git" \
          -H "Authorization: Basic $TOKEN" \
          -H "Content-Type: application/json"
        """
      }
    }

    stage('Release to Production') {
      steps {
        sh """
        curl -X POST "$BASE_URL/api/ext/apps/$APP_ID/git-sync/release" \
          -H "Authorization: Basic $TOKEN"
        """
      }
    }
  }
}
