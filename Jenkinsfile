pipeline {
    agent any

    environment {
        PROJECT_NAME = "react-cicd"
        DEPLOY_DIR   = "/var/www/html/react-cicd"
        DEVOPS_EMAIL = "krish.devopsmoweb@gmail.com"
    }

    triggers {
        githubPush()
    }

    tools {
        // Ensure "nodejs16" is the exact name in your Global Tool Configuration
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
                sh '''
                    echo "Current directory: $(pwd)"
                    npm ci --no-audit --no-fund || npm install --no-audit --no-fund
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    NODE_VER=$(node -v)
                    NODE_MAJOR=$(echo "$NODE_VER" | sed -E 's/^v([0-9]+).*/\\1/')
                    # Handles OpenSSL issues for Node 17-19
                    if [ "$NODE_MAJOR" -ge 17 ] && [ "$NODE_MAJOR" -le 19 ]; then
                        export NODE_OPTIONS=--openssl-legacy-provider
                    fi
                    npm run build
                '''
            }
        }

        stage('SonarQube analysis') {
            steps {
                // FIXED: Using 'sonarqube' (lowercase) to match your Jenkins System configuration screenshot
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        npx -y sonar-scanner \
                          -Dsonar.projectKey=${PROJECT_NAME} \
                          -Dsonar.sources=src \
                          -Dsonar.exclusions=node_modules/**,build/** \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_AUTH_TOKEN}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // This waits for the result from the SonarQube Webhook
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "❌ Quality Gate failed: ${qg.status}"
                    }
                }
            }
        }

        stage('Approval before deploy') {
            steps {
                emailext subject: "Approval required: deploy ${PROJECT_NAME}",
                         to: "${DEVOPS_EMAIL}",
                         body: """Build passed and Quality Gate is OK.
                         
                         Approve the deployment here: ${env.BUILD_URL}input"""
                
                input message: "Deploy ${PROJECT_NAME} to production?", ok: "Deploy"
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying build artifacts to ${DEPLOY_DIR}"
                    sudo mkdir -p ${DEPLOY_DIR}
                    # Clean the directory but keep the folder itself to preserve permissions
                    sudo rm -rf ${DEPLOY_DIR}/*
                    sudo cp -r build/* ${DEPLOY_DIR}/
                    sudo chown -R www-data:www-data ${DEPLOY_DIR}
                    echo "Deployment successful."
                '''
            }
        }
    }

    post {
        success {
            emailext subject: "✅ Deployment SUCCESS: ${PROJECT_NAME}",
                     to: "${DEVOPS_EMAIL}",
                     body: "The pipeline finished successfully. View build: ${env.BUILD_URL}"
        }
        failure {
            emailext subject: "❌ Pipeline FAILED: ${PROJECT_NAME}",
                     to: "${DEVOPS_EMAIL}",
                     body: "The pipeline failed. Check the logs here: ${env.BUILD_URL}"
        }
    }
}
