pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    environment {
        TARGET_DIR = "/var/www/html"
        SONAR_HOST = "http://192.168.31.252:9000"
        VM2_IP = "192.168.31.252"
        VM2_SSH_USER = "root"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/amittyagi1269/Project.git'
            }
        }

        stage('Code Analysis (SonarQube)') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        sonar-scanner \
                          -Dsonar.host.url=${SONAR_HOST} \
                          -Dsonar.token=${SONAR_TOKEN} \
                          -Dsonar.projectKey=Project-CI-CD \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=Jenkinsfile,plugins.txt,.git/**
                    '''
                }
            }
        }

        stage('Deploy to VM2 (Apache)') {
            steps {
                sh '''
                    SSH_CMD="ssh -o StrictHostKeyChecking=no"

                    $SSH_CMD ${VM2_SSH_USER}@${VM2_IP} "mkdir -p ${TARGET_DIR}"

                    rsync -avz --delete \
                      -e "$SSH_CMD" \
                      --exclude='.git' --exclude='Jenkinsfile' --exclude='plugins.txt' \
                      ./ ${VM2_SSH_USER}@${VM2_IP}:${TARGET_DIR}/

                    echo "Deployment to VM2 (${VM2_IP}:${TARGET_DIR}) completed successfully"
                '''
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ SUCCESS: Build #${env.BUILD_NUMBER} - ${env.JOB_NAME}",
                body: """<p>Build Status: <b>${currentBuild.currentResult}</b></p>
                         <p>Project: ${env.JOB_NAME}</p>
                         <p>Build Number: #${env.BUILD_NUMBER}</p>
                         <p>Duration: ${currentBuild.durationString}</p>
                         <p>Build URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>""",
                to: "${env.ALERT_EMAIL}",
                mimeType: 'text/html'
            )
        }
        failure {
            emailext (
                subject: "❌ FAILED: Build #${env.BUILD_NUMBER} - ${env.JOB_NAME}",
                body: """<p>Build Status: <b>${currentBuild.currentResult}</b></p>
                         <p>Project: ${env.JOB_NAME}</p>
                         <p>Build Number: #${env.BUILD_NUMBER}</p>
                         <p>Check the console output: <a href='${env.BUILD_URL}console'>${env.BUILD_URL}console</a></p>""",
                to: "${env.ALERT_EMAIL}",
                mimeType: 'text/html'
            )
        }
    }
}
