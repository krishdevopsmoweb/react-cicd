pipeline {
    agent any

    tools {
        nodejs 'node16'
    }

    environment {
        DEPLOY_DIR = "/var/www/krish.run.place"
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
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to Apache') {
            steps {
                sh '''
                  rsync -av --delete build/ ${DEPLOY_DIR}/
                  chown -R www-data:www-data ${DEPLOY_DIR}
                  chmod -R 755 ${DEPLOY_DIR}
                '''
            }
        }
    }
}
