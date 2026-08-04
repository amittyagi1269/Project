pipeline {
    agent any

    environment {
        VM2_IP = '10.109.35.198'
        VM2_USER = 'root'
        TARGET_DIR = '/var/www/html'
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                    sonar-scanner \
                    -Dsonar.host.url=http://f77:9000 \
                    -Dsonar.projectKey=Project-CI-CD-Pipeline \
                    -Dsonar.sources=. \
                    -Dsonar.token=$SONAR_TOKEN
                '''
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
            echo 'Deployment to VM2 and SonarQube analysis completed successfully!'
            emailext (
                subject: "SUCCESS: Pipeline Job '${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]'",
                body: "Good news! The CI/CD pipeline completed successfully after code update.\n\nTarget VM: ${env.VM2_IP}\nConsole Output: ${env.BUILD_URL}",
                to: "amittyagi1269@gmail.com"
            )
        }
        failure {
            echo 'Pipeline failed. Sending alert email...'
            emailext (
                subject: "FAILED: Pipeline Job '${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]'",
                body: "Oops! The CI/CD pipeline has failed during execution.\n\nCheck logs and fix issues at: ${env.BUILD_URL}",
                to: "amittyagi1269@gmail.com"
            )
        }
    }
}
