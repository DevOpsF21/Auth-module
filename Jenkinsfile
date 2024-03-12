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
        sh '/usr/local/bin/docker run -d --name auth-container -p 3000:3000 auth-module:latest'
      }
    }
    stage('Deploying to Minikube') {
      steps {
        sh '/usr/local/bin/kubectl apply -f deployment.yaml -f service.yaml'
      }
    }
  }
}
