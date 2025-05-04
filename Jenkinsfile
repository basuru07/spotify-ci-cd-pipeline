pipeline {
    agent any
    
    environment {
        // Define Docker Hub credentials - use your actual DockerHub username here
        DOCKER_USERNAME = "basuruyasaruwan"
        // The image name without username prefix (we'll add it in the commands)
        IMAGE_NAME = "spotify-clone"
        // Using Jenkins credential store
        DOCKER_CREDENTIALS = credentials('docker-credentials-id')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/basuru07/spotify-ci-cd-pipeline', branch: 'main'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_USERNAME}/${IMAGE_NAME}:latest --no-cache ."
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm install -g htmlhint || true'
                sh 'echo "Running basic validation checks"'
                sh 'htmlhint *.html || true'
            }
        }
        
        stage('Manual Docker Login') {
            steps {
                // This approach directly uses the credential variables
                sh '''
                    echo "Attempting manual Docker login..."
                    echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                '''
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                // Push using the fully qualified image name
                sh "docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:latest"
                sh "docker logout"
            }
        }
        
        stage('Deploy') {
            steps {
                // Stop and remove any existing container
                sh 'docker stop spotify-clone || true'
                sh 'docker rm spotify-clone || true'
                
                // Run the new container
                sh "docker run -d --name spotify-clone -p 8081:80 ${DOCKER_USERNAME}/${IMAGE_NAME}:latest"
                
                echo "Application deployed successfully at http://localhost:8081"
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
        always {
            // Clean up workspace but don't remove the running container
            cleanWs()
        }
    }
}