pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-credentials-id')
        IMAGE_NAME = "basuruyasaruwan/spotify-clone"
    }
    
    stages {
        stage('Checkout') {
            steps {
                bat 'git config --global --add safe.directory %CD%'
                git url: 'https://github.com/basuru07/spotify-ci-cd-pipeline', branch: 'main', credentialsId: 'github-token'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        bat "docker build -t %IMAGE_NAME%:latest --no-cache ."
                    } catch (Exception e) {
                        echo "Docker build failed: ${e.message}"
                        error "Build failed"
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                bat 'npm install -g htmlhint || exit /b 0'
                bat 'echo Running basic validation checks'
                bat 'htmlhint *.html || exit /b 0'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    bat 'docker --version'
                    bat 'docker ps -q || echo Docker daemon not running'
                    bat 'echo Using DockerHub username: %DOCKERHUB_CREDENTIALS_USR%'
                    bat 'if not defined DOCKERHUB_CREDENTIALS_PSW echo Credential password not set'
                    
                    try {
                        withCredentials([usernamePassword(credentialsId: 'docker-credentials-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            bat '''
                                echo Logging in to Docker Hub...
                                echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                                if %ERRORLEVEL% neq 0 (
                                    echo Docker login failed
                                    exit /b 1
                                )
                                echo Docker login successful
                                echo Pushing image %IMAGE_NAME%:latest to Docker Hub...
                                docker push %IMAGE_NAME%:latest
                                if %ERRORLEVEL% neq 0 (
                                    echo Docker push failed
                                    exit /b 1
                                )
                                echo Docker push successful
                                docker logout
                            '''
                        }
                    } catch (Exception e) {
                        echo "Full error details: ${e}"
                        error "Push to Docker Hub failed: ${e.message}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    try {
                        bat 'docker stop spotify-clone-container || exit /b 0'
                        bat 'docker rm spotify-clone-container || exit /b 0'
                        bat "docker run -d --name spotify-clone-container -p 8082:80 %IMAGE_NAME%:latest"
                        echo Application deployed successfully at http://localhost:8082
                    } catch (Exception e) {
                        echo "Deployment failed: ${e.message}"
                        error "Deploy failed"
                    }
                }
            }
        }
    }
    
    post {
        always {
            bat 'docker images -q | ForEach-Object { docker rmi $_ -f } || exit /b 0'
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