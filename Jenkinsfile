pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://192.168.112.132:2375'  // Replace <docker-server-ip> with your Docker server IP
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone your Git repository and specify the correct branch name
                git branch: 'main', url: 'https://github.com/marahman418/test-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image on the remote Docker server
                    sh 'docker -H $DOCKER_HOST build -t webserver .'
                }
            }
        }

        stage('Test Docker Container') {
            steps {
                script {
                    // Run the Docker container on the remote Docker server
                    sh 'docker -H $DOCKER_HOST run -d --name apache-test -p 8080:80 webserver'

                    // Test the container to check if it's running correctly
                    sh 'curl -I http://192.168.112.132:8080'  // Replace with your Docker server IP

                    // Stop and remove the test container after testing
                    sh 'docker -H $DOCKER_HOST stop apache-test && docker -H $DOCKER_HOST rm apache-test'
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    // Use Docker credentials to login and push the image to Docker registry
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                            docker -H $DOCKER_HOST login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                            docker -H $DOCKER_HOST tag webserver marahman418/webserver:latest
                            docker -H $DOCKER_HOST push marahman418/webserver:latest
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up dangling images to free up space on the Docker server
            script {
                sh 'docker -H $DOCKER_HOST image prune -f'
            }
        }
        failure {
            echo "Pipeline failed. Check the logs for more details."
        }
        success {
            echo "Pipeline completed successfully!"
        }
    }
}

