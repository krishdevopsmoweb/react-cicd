pipeline {
    agent any

    environment {
        DEPLOY_PATH = "/var/www/html/react-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/krishdevopsmoweb/react-cicd.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  node -v
                  npm install
                '''
            }
        }

        stage('Build React App') {
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
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Approval Before Deploy') {
            steps {
                emailext(
                    to: 'krish.devopsmoweb@gmail.com',
                    subject: "Approval Needed: React App Deployment",
                    body: """
                    <h3>Deployment Approval Required</h3>
                    <p>Project: React CI/CD</p>
                    <p>Build Number: ${BUILD_NUMBER}</p>
                    <p><b>Click Jenkins and approve deployment.</b></p>
                    """
                )

                input message: 'Do you want to deploy to production?',
                      ok: 'Deploy Now'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                  rm -rf ${DEPLOY_PATH}/*
                  cp -r build/* ${DEPLOY_PATH}/
                '''
            }
        }
    }

    post {
        success {
            emailext(
                to: 'krish.devopsmoweb@gmail.com',
                subject: "✅ Deployment Successful - Build #${BUILD_NUMBER}",
                body: "React app deployed successfully."
            )
        }

        failure {
            emailext(
                to: 'krish.devopsmoweb@gmail.com',
                subject: "❌ Pipeline Failed - Build #${BUILD_NUMBER}",
                body: "Pipeline failed. Please check Jenkins logs."
            )
        }
    }
}
