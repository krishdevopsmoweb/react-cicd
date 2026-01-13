pipeline {
    agent any

    environment {
        PROJECT_NAME = "react-cicd"
        DEPLOY_DIR   = "/var/www/html/react-app"
        DEVOPS_EMAIL = "krish.devopsmoweb@gmail.com"
    }

    triggers {
        githubPush()
    }

    tools {
        nodejs "nodejs16"
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
            environment {
                NODE_OPTIONS = "--openssl-legacy-provider"
            }
            steps {
                sh 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=react-cicd \
                        -Dsonar.sources=src
                    '''
                }
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
                    subject: "🚀 Approval Required: Deploy react-cicd",
                    to: "${DEVOPS_EMAIL}",
                    body: """
Build SUCCESS & Quality Gate PASSED ✅

Approve deployment:
${env.BUILD_URL}input
"""
                )
                input message: 'Deploy to Production?', ok: 'Deploy'
            }
        }

        stage('Deploy') {
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
                subject: "✅ Deployment SUCCESS: react-cicd",
                to: "${DEVOPS_EMAIL}",
                body: "Deployment completed successfully."
            )
        }

        failure {
            emailext(
                subject: "❌ Pipeline FAILED: react-cicd",
                to: "${DEVOPS_EMAIL}",
                body: "Check logs: ${env.BUILD_URL}"
            )
        }
    }
}

