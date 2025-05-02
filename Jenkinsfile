pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/basuru07/spotify-ci-cd-pipeline', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                script {
                    docker.build('spotify-app:latest')
                }
            }
        }
        stage('Test') {
            steps {
                sh 'echo "Running tests"'
                // Add test commands if you have a test suite (e.g., npm test)
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials-id') {
                        docker.image('spotify-app:latest').push('latest')
                    }
                }
            }
        }
    }
}