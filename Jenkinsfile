pipeline {
    agent any

    environment {
        SONAR_HOST_URL = "http://3.109.214.202:9000"
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/krishdevopsmoweb/react-cicd.git'
            }
        }

        stage('Use NodeJS') {
            steps {
                nodejs(nodeJSInstallationName: 'nodejs16') {
                    sh '''
                      node -v
                      npm -v
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                nodejs(nodeJSInstallationName: 'nodejs16') {
                    sh 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                nodejs(nodeJSInstallationName: 'nodejs16') {
                    sh 'npm run build'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    nodejs(nodeJSInstallationName: 'nodejs16') {
                        sh '''
                          npx sonar-scanner \
                          -Dsonar.projectKey=react-cicd \
                          -Dsonar.sources=src \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                  sudo rm -rf /var/www/krish.run.place/*
                  sudo cp -r build/* /var/www/krish.run.place/
                '''
            }
        }
    }
}
