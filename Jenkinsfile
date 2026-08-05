pipeline {
    agent any

    environment {
        VM2_IP = '10.43.7.198'
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
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh 'sonar-scanner -Dsonar.host.url=http://sonarqube:9000 -Dsonar.projectKey=my-project -Dsonar.sources=. -Dsonar.login=$SONAR_TOKEN'
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
            echo 'Deployment to VM2 and SonarQube analysis completed successfully!'
            emailext (
                subject: "SUCCESS: Pipeline Job '${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]'",
                body: "Good news! The CI/CD pipeline completed successfully.\n\nTarget VM: ${env.VM2_IP}\nConsole Output: ${env.BUILD_URL}",
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
