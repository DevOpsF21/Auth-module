pipeline {
    agent any
    
    stages {
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t auth-module .'
            }
        }
        stage('Push Docker Image to Local Registry') {
            steps {
                sh 'docker tag auth-module localhost:5000/auth-module' // Tagging the image for the local registry
                sh 'docker push localhost:5000/auth-module' // Pushing the image to the local registry
            }
        }
        stage('Run Docker Container') {
            steps {
                sh 'docker run -d --name auth-container -p 8080:8080 localhost:5000/auth-module' // Running the image from the local registry
            }
        }
        stage('Wait for Application to Start') {
            steps {
                // Add code to wait for the application to start (e.g., polling a health endpoint)
                sh 'sleep 30' // Example: Wait for 30 seconds
            }
        }
        stage('Run Tests') {
            steps {
                // Run Postman tests using Newman
                sh 'newman run <path_to_your_postman_collection>'
            }
        }
        stage('Cleanup') {
            steps {
                // Cleanup: Stop and remove the Docker container
                sh 'docker stop auth-container'
                sh 'docker rm auth-container'
            }
        }
    }
}

