pipeline {
  agent any
  stages {
    
    stage('Building Docker Image') {
      steps {
        sh '/usr/local/bin/docker build -t auth-module .'
      }
    }
      stage('Run Docker Container') {
             steps {
                 sh '/usr/local/bin/docker run -d --name auth-container -p 8080:8080 localhost:3000/auth-module'
             }
         }
    stage('Deploying Dockerized Application to Kubernetes') {
      steps {
        script {
          // Deploying the Dockerized application to the Kubernetes cluster
          kubernetesDeploy(configs: ["deployment.yaml", "service.yaml"])
        }
      }
    }
  }
}


