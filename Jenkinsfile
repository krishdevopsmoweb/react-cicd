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
                sh 'npm ci --no-audit --no-fund || npm install --no-audit --no-fund'
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
                withSonarQubeEnv('sonarqube') {
                    sh "npx -y sonar-scanner -Dsonar.projectKey=${PROJECT_NAME} -Dsonar.sources=src -Dsonar.exclusions=node_modules/**,build/**"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "❌ Quality Gate failed: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Approval') {
            steps {
                emailext subject: "Approval required: deploy ${PROJECT_NAME}",
                         to: "${DEVOPS_EMAIL}",
                         body: "Quality Gate OK. Approve here: ${env.BUILD_URL}input"
                
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
                     body: "Build successful: ${env.BUILD_URL}"
        }
        failure {
            emailext subject: "❌ Pipeline FAILED: ${PROJECT_NAME}",
                     to: "${DEVOPS_EMAIL}",
                     body: "Build failed: ${env.BUILD_URL}"
        }
    }
}
