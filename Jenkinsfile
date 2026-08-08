pipeline {
    agent any

    environment {
        VM2_IP = '192.168.31.252'
        VM2_USER = 'root'
        TARGET_DIR = '/var/www/html'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
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
            echo 'Deployment to VM2 (/var/www/html) completed successfully!'
        }
        failure {
            echo 'Pipeline failed during deployment.'
        }
    }
}
