pipeline {
  agent any

  tools {
    nodejs 'node20'
  }

  environment {
    DISCORD_WEBHOOK   = credentials('discord-webhook')
    RENDER_API_KEY    = credentials('render-api-key')
    RENDER_SERVICE_ID = 'srv-d5pi8d1r0fns73f28f9g'
  }

  options {
    timestamps()
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('SCA - npm audit') {
      steps {
        sh 'npm audit --audit-level=high'
      }
    }

    stage('SAST - Semgrep') {
      steps {
        sh '''
          set -e
          echo "Installation Semgrep..."
          python3 -m pip install --user semgrep
          export PATH="$HOME/.local/bin:$PATH"
          semgrep --config p/javascript .
        '''
      }
    }

    stage('Lint') {
      steps {
        sh 'npm run lint'
      }
    }

    stage('Tests') {
      steps {
        sh 'npm test'
      }
    }

    stage('Deploy Render') {
      when {
        branch 'main'
      }
      steps {
        sh '''
          echo "Déploiement Render..."
          curl -X POST \
          "https://api.render.com/deploy/${RENDER_SERVICE_ID}?key=${RENDER_API_KEY}"
        '''
      }
    }
  }

  post {

    success {
      sh '''
        curl -H "Content-Type: application/json" \
        -X POST \
        -d "{\"content\":\"✅ SUCCESS - ${JOB_NAME} #${BUILD_NUMBER}\"}" \
        "$DISCORD_WEBHOOK"
      '''
    }

    failure {
      sh '''
        curl -H "Content-Type: application/json" \
        -X POST \
        -d "{\"content\":\"❌ FAILURE - ${JOB_NAME} #${BUILD_NUMBER}\"}" \
        "$DISCORD_WEBHOOK"
      '''
    }

    always {
      echo "Pipeline terminée."
    }
  }
}
