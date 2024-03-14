pipeline {
  agent any
  environment {
    // Define Docker and Kubectl path as environment variables for easy changes
    DOCKER_CMD = '/usr/local/bin/docker'
    KUBECTL_CMD = '/usr/local/bin/kubectl'
  }
  stages {
    stage('Building Docker Image') {
      steps {
        script {
          // Building the Docker image
          sh "${env.DOCKER_CMD} build -t auth-module ."
        }
      }
    }
    stage('Run Docker Container Locally') {
      steps {
        script {
          // Stopping and removing the previous container if it exists
          sh "(${env.DOCKER_CMD} stop auth-container || true) && (${env.DOCKER_CMD} rm auth-container || true)"
          // Running the new Docker container
          sh "${env.DOCKER_CMD} run -d --name auth-container -p 3000:3000 auth-module:latest"
        }
      }
    }
    stage('Deploying to Minikube') {
      steps {
        script {
          // Applying Kubernetes configurations
          sh "${env.KUBECTL_CMD} apply -f deployment.yaml -f service.yaml"
        }
      }
    }
  }
  post {
    always {
      // Cleanup actions, such as notifying stakeholders or archiving artifacts, can go here
    }
  }
}
