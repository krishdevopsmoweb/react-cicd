pipeline {
    agent any

    tools {
        nodejs 'nodejs16'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
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
                        -Dsonar.host.url=http://3.109.214.202:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
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

        stage('Deploy to Apache') {
            steps {
                sh '''
                  sudo rm -rf /var/www/krish.run.place/*
                  sudo cp -r build/* /var/www/krish.run.place/
                  sudo chown -R www-data:www-data /var/www/krish.run.place
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }

        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
