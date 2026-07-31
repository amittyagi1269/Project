pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://sonarqube:9000'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Code Testing') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        echo "Running automated SonarQube code scan..."
                        sonar-scanner \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.token=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Deploy to VM2') {
            steps {
                sh '''
                    echo "Deploying code to VM2 via SSH..."
                    rsync -avz -e "ssh -o StrictHostKeyChecking=no" --exclude=.git ./ root@10.26.0.198:/var/www/html/
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed and deployed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
