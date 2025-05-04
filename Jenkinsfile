pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-credentials-id')
        IMAGE_NAME = "basuruyasaruwan/spotify-clone"  // Fixed image name as requested
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
                        sh "docker build -t ${IMAGE_NAME}:latest --no-cache ."
                    } catch (Exception e) {
                        error "Docker build failed: ${e.message}"
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm install -g htmlhint || true' // Install htmlhint if not present
                sh 'echo "Running basic validation checks"'
                sh 'htmlhint *.html || true' // Validate HTML files
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    try {
                        // Login to Docker Hub
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        
                        // Push the image
                        sh "docker push ${IMAGE_NAME}:latest"
                        
                        // Logout for security
                        sh 'docker logout'
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
                        // Stop and remove any existing container with the specified name
                        sh 'docker stop spotify-clone || true'
                        sh 'docker rm spotify-clone || true'
                        
                        // Run the new container with the correct name
                        sh "docker run -d --name spotify-clone -p 8081:80 ${IMAGE_NAME}:latest"
                        
                        echo "Application deployed successfully at http://localhost:8081"
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
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}