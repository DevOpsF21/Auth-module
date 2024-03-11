pipeline {
    agent any
    
    stages {
        stage('Install Dependencies') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            tools {
                nodejs 'NodeJS 21' 
            }
            steps {
                sh 'npm install'
            }
        }
        stage('Build Docker Image') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            steps {
                sh 'docker build -t auth-module .'
            }
        }
        stage('Push Docker Image to Local Registry') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            steps {
                sh 'docker tag auth-module localhost:5000/auth-module' // Tagging the image for the local registry
                sh 'docker push localhost:5000/auth-module' // Pushing the image to the local registry
            }
        }
        stage('Run Docker Container') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            steps {
                sh 'docker run -d --name auth-container -p 8080:8080 localhost:5000/auth-module' // Running the image from the local registry
            }
        }
        stage('Wait for Application to Start') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            steps {
                // Add code to wait for the application to start (e.g., polling a health endpoint)
                sh 'sleep 30' // Example: Wait for 30 seconds
            }
        }
        stage('Run Tests') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            steps {
                // Run Postman tests using Newman
                sh 'newman run <path_to_your_postman_collection>'
            }
        }
        stage('Cleanup') {
            agent {
                label 'docker' // Set the agent label to specify where this stage runs
            }
            steps {
                // Cleanup: Stop and remove the Docker container
                sh 'docker stop auth-container'
                sh 'docker rm auth-container'
            }
        }
    }
}
