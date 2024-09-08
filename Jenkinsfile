pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://192.168.112.132:2375'  // Docker server IP
        KUBECONFIG = '/var/lib/jenkins/.kube/config'  // Path to kubeconfig file on Jenkins
        DOCKER_IMAGE = 'marahman418/webserver:latest'  // Docker Hub image
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/marahman418/test-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker -H $DOCKER_HOST build -t webserver .'
            }
        }

        stage('Test Docker Container') {
            steps {
                sh 'docker -H $DOCKER_HOST run -d --name apache-test -p 8080:80 webserver'
                sh 'curl -I http://192.168.112.132:8080'
                sh 'docker -H $DOCKER_HOST stop apache-test'
                sh 'docker -H $DOCKER_HOST rm apache-test'
            }
        }

        stage('Push to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh """
                        docker -H $DOCKER_HOST login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker -H $DOCKER_HOST tag webserver marahman418/webserver:latest
                        docker -H $DOCKER_HOST push marahman418/webserver:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Create or update the deployment using kubectl
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG apply -f - <<EOF
                        apiVersion: apps/v1
                        kind: Deployment
                        metadata:
                          name: webserver
                          labels:
                            app: webserver
                        spec:
                          replicas: 2
                          selector:
                            matchLabels:
                              app: webserver
                          template:
                            metadata:
                              labels:
                                app: webserver
                            spec:
                              containers:
                              - name: webserver
                                image: $DOCKER_IMAGE
                                ports:
                                - containerPort: 80
                        EOF
                    """

                    // Expose the deployment as a service
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG expose deployment webserver --type=LoadBalancer --name=webserver-service --port=80
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker -H $DOCKER_HOST image prune -f'
        }
        failure {
            echo "Pipeline failed. Check the logs for more details."
        }
        success {
            echo "Pipeline completed successfully!"
        }
    }
}




