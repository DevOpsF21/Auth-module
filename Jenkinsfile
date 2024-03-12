pipeline {
  agent any
  stages {
    stage('Building Docker Image') {
      steps {
        sh '/usr/local/bin/docker build -t auth-module .'
      }
    }
    stage('Run Docker Container Locally') {
      steps {
        sh '/usr/local/bin/docker run -d --name auth-container -p 8080:8080 auth-module:latest'
      }
    }
    stage('Deploying to Minikube') {
      steps {
        sh 'kubectl apply -f deployment.yaml -f service.yaml'
      }
    }
  }
}
