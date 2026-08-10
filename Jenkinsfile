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

        stage('Deploy Locally (Apache on VM)') {
            steps {
                sh '''
                    # Ensure target directory exists
                    sudo mkdir -p ${TARGET_DIR}

                    # Sync workspace directly to local VM Apache web root
                    sudo rsync -avz --delete \
                      --exclude='.git' \
                      --exclude='Jenkinsfile' \
                      --exclude='plugins.txt' \
                      ./ ${TARGET_DIR}/

                    # Fix permissions for Apache (www-data for Ubuntu/Debian, apache for RHEL/CentOS)
                    #sudo chown -R www-data:www-data ${TARGET_DIR} || sudo chown -R apache:apache ${TARGET_DIR}
                    #sudo chmod -R 755 ${TARGET_DIR}

                    # Reload web server service
                    sudo systemctl reload apache2 || sudo systemctl reload httpd || true

                    echo "Local VM deployment to ${TARGET_DIR} completed successfully!"
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
