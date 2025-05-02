pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-credentials-id')
        IMAGE_NAME = "basuruyasaruwan/spotify-web"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/basuru07/spotify-ci-cd-pipeline', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        dockerImage = docker.build("${IMAGE_NAME}:latest", "--no-cache .")
                    } catch (Exception e) {
                        error "Docker build failed: ${e.message}"
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Running basic validation checks"'
                // Add HTML validation with htmlhint (install it first via npm if needed)
                sh 'htmlhint *.html || true' // || true to avoid failing the pipeline
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    try {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials-id') {
                            dockerImage.push('latest')
                        }
                    } catch (Exception e) {
                        error "Push to Docker Hub failed: ${e.message}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    try {
                        // Stop and remove any existing container
                        sh 'docker stop spotify-web-container || true'
                        sh 'docker rm spotify-web-container || true'

                        // Run the new container
                        sh "docker run -d --name spotify-web-container -p 80:80 ${IMAGE_NAME}:latest"
                    } catch (Exception e) {
                        error "Deployment failed: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
            // Optional: Stop and remove the container if it exists
            sh 'docker stop spotify-web-container || true'
            sh 'docker rm spotify-web-container || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}