pipeline {
    agent any

    tools {
        nodejs 'nodejs16'
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
                  cd /var/www/html/react-cicd
                  node -v
                  npm -v
                  npm install
                '''
            }
        }

        stage('Build React App') {
            steps {
                sh '''
                  cd /var/www/html/react-cicd
                  npm run build
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      cd /var/www/html/react-cicd
                      npx sonar-scanner
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Waiting for SonarQube Quality Gate..."
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Deploy (Build → Apache)') {
            steps {
                sh '''
                  echo "Deploying React build to Apache DocumentRoot"
                  cd /var/www/html/react-cicd

                  rm -rf public static asset-manifest.json favicon.ico index.html manifest.json robots.txt

                  cp -r build/* .
                '''
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Build passed Quality Gate and deployed"
        }
        failure {
            echo "❌ FAILURE: Quality Gate failed or build error"
        }
    }
}
