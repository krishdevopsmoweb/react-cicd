pipeline {
    agent any

    environment {
        // Fix OpenSSL issue for webpack/react-scripts
        NODE_OPTIONS = "--openssl-legacy-provider"

        // Project details
        PROJECT_NAME = "react-cicd"
        DEPLOY_DIR   = "/var/www/html/react-app"

        // SonarQube
        SONAR_PROJECT_KEY = "react-cicd"
        SONAR_HOST_URL    = "http://3.109.214.202:9000"
        SONAR_TOKEN       = credentials('sonar-token')

        // Email
        DEVOPS_EMAIL = "krish.devopsmoweb@gmail.com"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    node -v
                    npm -v
                    npm install
                '''
            }
        }

        stage('Build React App') {
            steps {
                sh '''
                    npm run build
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    sonar-scanner \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.sources=src \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "❌ Quality Gate failed: ${qg.status}"
                    }
                }
            }
        }

        stage('Approval Before Deploy') {
            steps {
                emailext(
                    subject: "🚀 Approval Required: Deploy ${PROJECT_NAME}",
                    to: "${DEVOPS_EMAIL}",
                    body: """
Build SUCCESS and Quality Gate PASSED ✅

Project: ${PROJECT_NAME}
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}

👉 Please approve deployment in Jenkins:
${env.BUILD_URL}input
"""
                )

                input message: 'Do you want to deploy to PRODUCTION?', ok: 'Deploy'
            }
        }

        stage('Deploy to Apache') {
            steps {
                sh '''
                    sudo rm -rf ${DEPLOY_DIR}
                    sudo mkdir -p ${DEPLOY_DIR}
                    sudo cp -r build/* ${DEPLOY_DIR}/
                    sudo chown -R www-data:www-data ${DEPLOY_DIR}
                '''
            }
        }
    }

    post {

        success {
            emailext(
                subject: "✅ Deployment SUCCESS: ${PROJECT_NAME}",
                to: "${DEVOPS_EMAIL}",
                body: """
Deployment completed successfully 🎉

Project: ${PROJECT_NAME}
Build: #${env.BUILD_NUMBER}

Live on server.
"""
            )
        }

        failure {
            emailext(
                subject: "❌ Pipeline FAILED: ${PROJECT_NAME}",
                to: "${DEVOPS_EMAIL}",
                body: """
Pipeline FAILED ❌

Project: ${PROJECT_NAME}
Build: #${env.BUILD_NUMBER}

Check logs:
${env.BUILD_URL}
"""
            )
        }
    }
}

