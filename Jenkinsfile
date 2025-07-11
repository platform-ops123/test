pipeline {
  agent any

  environment {
    BASE_URL = credentials('TOOLJET_BASE_URL')
    TOKEN = credentials('TOOLJET_ACCESS_TOKEN')
    APP_ID = '64b94f2c-ec13-47c6-80bf-912dc940206a'
    VERSION_ID = 'v1'
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
