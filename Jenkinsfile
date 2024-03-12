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
        sh '/usr/local/bin/docker run -d --name auth-container -p 8080:8080 auth-module:latest'
      }
    }
    stage('Deploying Dockerized Application to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: ["deployment.yaml", "service.yaml"])
        }
      }
    }
  }
}
