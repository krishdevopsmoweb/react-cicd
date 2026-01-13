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
        // Recommendation: Update "nodejs16" to "nodejs18" or "nodejs20" in Global Tool Configuration
        // if your dependencies continue to throw warnings.
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
                    echo "Working dir: $(pwd)"
                    npm ci --no-audit --no-fund || npm install --no-audit --no-fund
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    NODE_VER=$(node -v)
                    NODE_MAJOR=$(echo "$NODE_VER" | sed -E 's/^v([0-9]+).*/\\1/')
                    if [ "$NODE_MAJOR" -ge 17 ] && [ "$NODE_MAJOR" -le 19 ]; then
                        export NODE_OPTIONS=--openssl-legacy-provider
                    fi
                    npm run build
                '''
            }
        }

        stage('SonarQube analysis') {
            steps {
                // Ensure the name 'SonarQube' matches exactly in Manage Jenkins -> System
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        npx -y sonar-scanner \
                          -Dsonar.projectKey=${PROJECT_NAME} \
                          -Dsonar.sources=src \
                          -Dsonar.exclusions=node_modules/**,build/** \
                          -Dsonar.projectBaseDir=$(pwd) \
                          -Dsonar.host.url=${SONAR_HOST_URL}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // This waits for SonarQube to finish processing and report back
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "❌ Quality Gate failed: ${qg.status}"
                    }
                }
            }
        }

        stage('Approval') {
            steps {
                emailext subject: "Approval required: deploy ${PROJECT_NAME}",
                         to: "${DEVOPS_EMAIL}",
                         body: "Build passed & Quality Gate OK. Approve here: ${env.BUILD_URL}input"
                
                input message: "Deploy ${PROJECT_NAME} to production?", ok: "Deploy"
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying to ${DEPLOY_DIR}"
                    sudo mkdir -p ${DEPLOY_DIR}
                    sudo rm -rf ${DEPLOY_DIR}/*
                    sudo cp -r build/* ${DEPLOY_DIR}/
                    sudo chown -R www-data:www-data ${DEPLOY_DIR}
                '''
            }
        }
    }

    post {
        success {
            emailext subject: "✅ Deployment SUCCESS: ${PROJECT_NAME}",
                     to: "${DEVOPS_EMAIL}",
                     body: "Deployment completed successfully: ${env.BUILD_URL}"
        }
        failure {
            emailext subject: "❌ Pipeline FAILED: ${PROJECT_NAME}",
                     to: "${DEVOPS_EMAIL}",
                     body: "Pipeline failed. Check logs: ${env.BUILD_URL}"
        }
    }
}
