pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'SonarScanner' // Ensure you configure this tool name in Jenkins Global Tool Configuration if needed, or invoke via docker
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'git-creds', url: 'https://github.com/amittyagi1269/Project.git'
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Running SonarQube analysis via Docker container or CLI
                    sh '''
                    docker run --rm -v $(pwd):/usr/src sonarsource/sonar-scanner-cli \
                      -Dsonar.projectKey=project1 \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://http://10.226.206.198:9000 \
                      -Dsonar.login=squ_683b6610486a58c280b63e18bf8c948e6e4f6228
                    '''
                }
            }
        }


        stage('3. Deploy to VM 2 (Apache)') {
            steps {
                sshagent(['vm2-ssh-key']) {
                    sh '''
                        // Compress code and transfer to VM 2 web directory securely via rsync/scp
                        tar -czf project1.tar.gz *
                        scp -o StrictHostKeyChecking=no project1.tar.gz root@10.226.206.198:/tmp/
                        ssh -o StrictHostKeyChecking=no root@10.226.206.198 "sudo tar -xzf /tmp/project1.tar.gz -C /var/www/html/ && sudo chown -R www-data:www-data /var/www/html/"
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "SUCCESS: Jenkins Pipeline Passed - project1",
                body: "The build and deployment for project1 was successful. Code is now live on VM 2.",
                to: "amittyagi1269@gmail.com"
            )
        }
        failure {
            emailext (
                subject: "FAILURE: Jenkins Pipeline Failed - project1",
                body: "The build or deployment for project1 failed. Please check the Jenkins console logs.",
                to: "amittyagi1269@gmail.com"
            )
        }
    }
}
