pipeline {
    agent any

    environment {
        VM2_IP = '192.168.31.252'
        VM2_USER = 'root'
        TARGET_DIR = '/var/www/html'
        SCANNER_HOME = tool 'SonarQubeScanner' 
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh 'echo "Token check (should not be empty): ${env.SONAR_AUTH_TOKEN}"'
                withSonarQubeEnv('SonarQubeServer') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=my-project \
                        -Dsonar.sources=. \
                        -Dsonar.token=${env.SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to VM2') {
            steps {
                sh """
                    echo 'Deploying code to VM2 via passwordless SSH...'
                    rsync -avz -e 'ssh -o StrictHostKeyChecking=no' --exclude='.git' ./ ${VM2_USER}@${VM2_IP}:${TARGET_DIR}/
                    ssh -o StrictHostKeyChecking=no ${VM2_USER}@${VM2_IP} 'systemctl reload httpd'
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully: Code tested via SonarQube and deployed to VM2!'
        }
        failure {
            echo 'Pipeline failed during execution, analysis, or deployment.'
        }
    }
}
