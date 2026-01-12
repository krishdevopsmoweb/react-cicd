pipeline {
    agent any

    tools {
        nodejs 'nodejs16'
    }

    environment {
        PROJECT_DIR = '/var/www/html/react-cicd'
        BUILD_DIR   = '/var/www/html/react-cicd/build'
    }

    stages {

        stage('Checkout SCM') {
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
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      npx sonar-scanner \
                        -Dsonar.projectKey=react-cicd \
                        -Dsonar.projectName=react-cicd \
                        -Dsonar.sources=src \
                        -Dsonar.exclusions=node_modules/**,build/**
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy (Build → Apache)') {
            steps {
                sh '''
                  echo "Deploying React build"

                  rm -rf ${BUILD_DIR}
                  cp -r build ${PROJECT_DIR}/

                  chown -R root:root ${PROJECT_DIR}
                  chmod -R 755 ${PROJECT_DIR}
                '''
            }
        }
    }

    post {
        success {
            echo '✅ React app deployed successfully'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
