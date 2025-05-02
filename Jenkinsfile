pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-credentials-id')
        IMAGE_NAME = "your-dockerhub-username/spotify-web"
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
                    dockerImage = docker.build("${IMAGE_NAME}:latest", "--no-cache .")
                }
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Running basic validation checks"'
                // Optional: Add HTML/CSS/JS linting (e.g., using htmlhint)
                // Example: sh 'htmlhint *.html' (requires htmlhint installation)
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials-id') {
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Stop and remove any existing container
                    sh 'docker stop spotify-web-container || true'
                    sh 'docker rm spotify-web-container || true'

                    // Run the new container
                    sh "docker run -d --name spotify-web-container -p 80:80 ${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}