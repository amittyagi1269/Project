pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    environment {
        TARGET_DIR  = "/var/www/html"
        SONAR_HOST  = "http://192.168.31.252:9000"
        ALERT_EMAIL = "amittyagi1269@gmail.com"
        REMOTE_HOST = "192.168.31.100" // Replace with your remote VM IP
        REMOTE_USER = "ubuntu"         // Replace with your remote VM user (e.g., ubuntu, centos, root)
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

        stage('Deploy to Remote VM') {
            steps {
                sshagent(credentialsId: 'remote-vm-ssh-key') {
                    sh '''
                        # Sync workspace files to the remote VM via rsync over SSH
                        rsync -avz --delete -e "ssh -o StrictHostKeyChecking=no" \
                          --exclude='.git' \
                          --exclude='Jenkinsfile' \
                          --exclude='plugins.txt' \
                          ./ ${REMOTE_USER}@${REMOTE_HOST}:${TARGET_DIR}/

                        # Fix ownership and restart the web server on the remote VM
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} \
                          "sudo chown -R www-data:www-data ${TARGET_DIR} || sudo chown -R apache:apache ${TARGET_DIR} && \
                           sudo chmod -R 755 ${TARGET_DIR} && \
                           (sudo systemctl reload apache2 || sudo systemctl reload httpd || true)"

                        echo "Remote deployment to ${REMOTE_USER}@${REMOTE_HOST}:${TARGET_DIR} completed successfully!"
                    '''
                }
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
