pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = '2246c8b9-3997-40b9-bbf3-59b2757ab0bd'  // Replace with your actual credentials ID
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        FRONTEND_IMAGE = 'anjali2454/frontend'
        BACKEND_IMAGE = 'anjali2454/backend'
        PORT_FRONTEND = '80'
        PORT_BACKEND = '3000'
        UNIQUE_SUFFIX = "${env.BUILD_ID}".replace('/', '_')  // Sanitize the UNIQUE_SUFFIX to avoid invalid characters
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout code from the repository
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                script {
                    // Build Docker images with the unique tag
                    sh "docker build -t ${env.FRONTEND_IMAGE}:${env.UNIQUE_SUFFIX} ./frontend"
                    sh "docker build -t ${env.BACKEND_IMAGE}:${env.UNIQUE_SUFFIX} ./backend"
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    docker.withRegistry(env.DOCKER_REGISTRY, env.DOCKER_CREDENTIALS_ID) {
                        sh 'echo "Logged in to Docker Hub"'
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    // Push Docker images with unique tag
                    docker.withRegistry(env.DOCKER_REGISTRY, env.DOCKER_CREDENTIALS_ID) {
                        sh "docker push ${env.FRONTEND_IMAGE}:${env.UNIQUE_SUFFIX}"
                        sh "docker push ${env.BACKEND_IMAGE}:${env.UNIQUE_SUFFIX}"

                        // Tag the images with 'latest'
                        sh "docker tag ${env.FRONTEND_IMAGE}:${env.UNIQUE_SUFFIX} ${env.FRONTEND_IMAGE}:latest"
                        sh "docker tag ${env.BACKEND_IMAGE}:${env.UNIQUE_SUFFIX} ${env.BACKEND_IMAGE}:latest"

                        // Push the 'latest' tag
                        sh "docker push ${env.FRONTEND_IMAGE}:latest"
                        sh "docker push ${env.BACKEND_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy Containers') {
            steps {
                script {
                    try {
                        // Deploy new containers using the 'latest' tag
                        sh """
                        docker ps -q --filter "name=${env.FRONTEND_IMAGE.replace('/', '_')}-" | xargs -r docker stop
                        docker ps -a -q --filter "name=${env.FRONTEND_IMAGE.replace('/', '_')}-" | xargs -r docker rm
                        docker ps -q --filter "name=${env.BACKEND_IMAGE.replace('/', '_')}-" | xargs -r docker stop
                        docker ps -a -q --filter "name=${env.BACKEND_IMAGE.replace('/', '_')}-" | xargs -r docker rm
                        
                        docker run -d --name ${env.FRONTEND_IMAGE.replace('/', '_')}-latest -p ${env.PORT_FRONTEND}:${env.PORT_FRONTEND} ${env.FRONTEND_IMAGE}:latest
                        docker run -d --name ${env.BACKEND_IMAGE.replace('/', '_')}-latest -p ${env.PORT_BACKEND}:${env.PORT_BACKEND} ${env.BACKEND_IMAGE}:latest
                        """
                    } catch (Exception e) {
                        error "Deployment failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up actions or notifications
            echo 'Pipeline completed. Images pushed with version and latest tags, and containers deployed.'
        }
        success {
            echo 'Pipeline succeeded. Images pushed with version and latest tags, and containers deployed.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
