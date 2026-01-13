pipeline {
  agent any

  environment {
    PROJECT_NAME = "react-cicd"
    DEPLOY_DIR   = "/var/www/html/react-cicd"   // <- your actual site location
    DEVOPS_EMAIL = "krish.devopsmoweb@gmail.com"
  }

  triggers {
    githubPush()
  }

  tools {
    // Make sure this exact installation name exists in Jenkins (Manage Jenkins → Global Tool Configuration)
    nodejs "nodejs16"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies') {
      steps {
        // run in workspace (Jenkins owns the workspace). Avoid building inside /var/www to prevent perms issues.
        sh '''
          echo "Working dir: $(pwd)"
          node -v
          npm -v
          npm ci --no-audit --no-fund || npm install --no-audit --no-fund
        '''
      }
    }

    stage('Build') {
      steps {
        // Only set the legacy OpenSSL option for Node 17..19 (where it's needed). Do NOT set it for Node 20+.
        sh '''
          NODE_VER=$(node -v)
          echo "node version: $NODE_VER"
          NODE_MAJOR=$(echo "$NODE_VER" | sed -E 's/^v([0-9]+).*/\\1/')
          if [ "$NODE_MAJOR" -ge 17 ] && [ "$NODE_MAJOR" -le 19 ]; then
            echo "Using legacy OpenSSL provider for Node $NODE_MAJOR"
            export NODE_OPTIONS=--openssl-legacy-provider
          else
            echo "Not setting NODE_OPTIONS for Node $NODE_MAJOR"
          fi
          npm run build
        '''
      }
    }

    stage('SonarQube analysis') {
      steps {
        // Requires SonarQube server configuration in Jenkins (Manage Jenkins -> Configure System -> SonarQube servers)
        // The block below uses withSonarQubeEnv so credentials are injected by the plugin.
        withSonarQubeEnv('SonarQube') {
          sh '''
            echo "SONAR_HOST_URL=$SONAR_HOST_URL"
            # run sonar-scanner via npx so you don't need a system-wide binary
            npx -y sonar-scanner \
              -Dsonar.projectKey=${PROJECT_NAME} \
              -Dsonar.sources=src \
              -Dsonar.exclusions=node_modules/**,build/** \
              -Dsonar.projectBaseDir=$(pwd)
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          // Wait for SonarQube Quality Gate status (no timeout — you asked to not timeout)
          def qg = waitForQualityGate()
          echo "Quality Gate status: ${qg.status}"
          if (qg.status != 'OK') {
            error "❌ Quality Gate failed: ${qg.status}"
          }
        }
      }
    }

    stage('Approval before deploy') {
      steps {
        // Send a notification email asking for approval (Jenkins email plugin must be configured)
        emailext subject: "Approval required: deploy ${PROJECT_NAME}",
                to: "${DEVOPS_EMAIL}",
                body: """Build passed & Quality Gate OK.

Approve deployment here:
${env.BUILD_URL}input
"""
        // interactive approval in Jenkins UI
        input message: "Deploy ${PROJECT_NAME} to production?", ok: "Deploy"
      }
    }

    stage('Deploy') {
      steps {
        // Copy build output to your real site dir. Jenkins must have sudo rights for these commands or run Jenkins as a user that can write there.
        sh '''
          echo "Deploying build -> ${DEPLOY_DIR}"
          sudo rm -rf ${DEPLOY_DIR} || true
          sudo mkdir -p ${DEPLOY_DIR}
          sudo cp -r build/* ${DEPLOY_DIR}/
          sudo chown -R www-data:www-data ${DEPLOY_DIR}
          echo "Deployed."
        '''
      }
    }
  } // stages

  post {
    success {
      emailext subject: "✅ Deployment SUCCESS: ${PROJECT_NAME}",
              to: "${DEVOPS_EMAIL}",
              body: "Deployment completed successfully: ${env.BUILD_URL}"
    }
    failure {
      emailext subject: "❌ Pipeline FAILED: ${PROJECT_NAME}",
              to: "${DEVOPS_EMAIL}",
              body: "Pipeline failed. See logs: ${env.BUILD_URL}"
    }
  }
}
