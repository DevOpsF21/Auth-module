pipeline {
    agent any
    
    stages {
     
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t auth-module .'
            }
        }
        stage('Push Docker Image to Local Registry') {
            steps {
                sh 'docker tag auth-module localhost:5000/auth-module'
                sh 'docker push localhost:5000/auth-module'
            }
        }
        stage('Run Docker Container') {
            steps {
                sh 'docker run -d --name auth-container -p 8080:8080 localhost:5000/auth-module'
            }
        }
        stage('Wait for Application to Start') {
            steps {
                sh 'sleep 30'
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
                sh 'docker stop auth-container'
                sh 'docker rm auth-container'
            }
        }
    }
}
