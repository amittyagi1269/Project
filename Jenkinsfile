pipeline {
    agent any

    environment {
        DEPLOY_SERVER = "root1@192.168.31.252"
        TARGET_DIR   = "/var/www/html"
        SONAR_HOST   = "http://192.168.31.252:9000"
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
                          -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Deploy to Target Server') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} "mkdir -p ${TARGET_DIR}"
                    rsync -avz -e "ssh -o StrictHostKeyChecking=no" --exclude='.git*' ./ ${DEPLOY_SERVER}:${TARGET_DIR}/
                '''
            }
        }
    }

    post {
        success {
            emailext (
                subject: "SUCCESSFUL: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
                body: """<p>Build <b>#${env.BUILD_NUMBER}</b> completed successfully.</p>
                         <p>URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>""",
                to: "amittyagi1269@gmail.com",
                mimeType: 'text/html'
            )
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
                body: """<p>Build <b>#${env.BUILD_NUMBER}</b> failed.</p>
                         <p>Check logs at: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>""",
                to: "amittyagi1269@gmail.com",
                mimeType: 'text/html'
            )
        }
    }
}
