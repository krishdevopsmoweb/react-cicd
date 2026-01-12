pipeline {
    agent any

    tools {
        nodejs 'nodejs16'
    }

    environment {
        DEPLOY_DIR = '/var/www/krish.run.place'
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

        stage('Deploy to Apache') {
            steps {
                sh '''
                  echo "Deploying to ${DEPLOY_DIR}"

                  rm -rf ${DEPLOY_DIR}/*
                  cp -r build/* ${DEPLOY_DIR}/

                  chown -R jenkins:www-data ${DEPLOY_DIR}
                  chmod -R 755 ${DEPLOY_DIR}
                '''
            }
        }
    }

    post {
        success {
            echo '✅ React app deployed successfully to krish.run.place'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}
