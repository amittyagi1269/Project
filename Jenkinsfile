pipeline {
    agent any

    environment {
        VM2_IP = '192.168.31.252'
        VM2_USER = 'root'
        TARGET_DIR = '/var/www/html'
        // Use credentials ID configured in Jenkins for secure token/auth handling
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
                // Utilizing Jenkins SonarQube scanner environment integration
                withSonarQubeEnv('SonarQubeServer') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=my-project \
                        -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Pauses pipeline until SonarQube analyzes and returns the Quality Gate status
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
