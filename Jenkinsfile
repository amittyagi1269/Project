pipeline {
    agent any

    environment {
        VM2_IP = '192.168.31.252'
        VM2_USER = 'root'
        TARGET_DIR = '/var/www/html'
        SCANNER_HOME = tool 'SonarQubeScanner' 
        // Directly bind your newly created Jenkins credential here:
        SONAR_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Project-CI-CD-Pipeline \
                        -Dsonar.sources=. \
                        -Dsonar.token=${SONAR_TOKEN}
                    '''
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
                sh '''
                    echo 'Deploying code to VM2 via passwordless SSH...'
                    rsync -avz -e 'ssh -o StrictHostKeyChecking=no' --exclude='.git' ./ ''' + "${VM2_USER}@${VM2_IP}:${TARGET_DIR}/" + '''
                    ssh -o StrictHostKeyChecking=no ''' + "${VM2_USER}@${VM2_IP}" + ''' 'systemctl reload httpd'
                '''
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
