pipeline {
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        git 'git@github.com:DevOpsF21/Auth-module.git'
      }
    }
    stage('Building Docker Image') {
      steps {
        sh '/usr/local/bin/docker build -t auth-module .'
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


