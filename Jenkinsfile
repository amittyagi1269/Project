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

        stage('SonarQube Analysis') {
            steps {
                sh 'sonar-scanner -Dsonar.host.url=http://192.168.31.252:9000 -Dsonar.projectKey=my-project -Dsonar.sources=.'
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
            echo 'Deployment to VM2 (/var/www/html) and SonarQube analysis completed successfully!'
            emailext (
                subject: "SUCCESS: Job '${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]'",
                body: "Great news! Your pipeline finished successfully.\n\nConsole output: ${env.BUILD_URL}",
                to: 'amittyagi1269@gmail.com',
                from: 'amittyagi1269@gmail.com'
            )
        }
        failure {
            echo 'Pipeline failed.'
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]'",
                body: "Alert: Your build or deployment has failed.\n\nConsole output: ${env.BUILD_URL}",
                to: 'amittyagi1269@gmail.com',
                from: 'amittyagi1269@gmail.com'
            )
        }
    }
}
