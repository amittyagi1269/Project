pipeline {
    agent any

    environment {
        VM2_IP = '10.59.244.198'
        VM2_USER = 'root'
        TARGET_DIR = '/var/www/html'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh 'sonar-scanner -Dsonar.host.url=http://10.59.244.198:9000 -Dsonar.projectKey=my-project -Dsonar.sources=.'
            }
        }

        stage('Deploy to VM2') {
            steps {
                sh """
                    echo 'Deploying code to VM2 via passwordless SSH...'
                    rsync -avz -e 'ssh -o StrictHostKeyChecking=no' --exclude='.git' ./ ${VM2_USER}@${VM2_IP}:${TARGET_DIR}/
                    ssh -o StrictHostKeyChecking=no ${VM2_USER}@${VM2_IP} 'systemctl reload apache2'
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment to VM2 (/var/www/html) and SonarQube analysis completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
