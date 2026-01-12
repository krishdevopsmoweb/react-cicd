pipeline {
    agent any

    tools {
        nodejs 'nodejs16'   // MUST match Jenkins → Manage Tools
    }

    environment {
        SONAR_PROJECT_KEY = 'react-cicd'
        SONAR_PROJECT_NAME = 'react-cicd'
        DEPLOY_DIR = '/var/www/html/react-app'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/krishdevopsmoweb/react-cicd.git'
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
                sh 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      npx sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=src \
                        -Dsonar.exclusions=node_modules/**,build/** \
                        -Dsonar.host.url=http://3.109.214.202:9000 \
                        -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
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
            echo '✅ CI/CD Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
        aborted {
            echo '⚠️ Pipeline aborted (timeout or manual stop).'
        }
    }
}
