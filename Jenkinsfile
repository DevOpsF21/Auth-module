pipeline {
  agent any
  environment {
    DOCKER_IMAGE = 'auth-module:latest'
    MINIKUBE_PROFILE = 'minikube'
  }
  stages {
    stage('Building Docker Image') {
      steps {
        script {
          // Build the Docker image
          sh '/usr/local/bin/docker build -t ${DOCKER_IMAGE} .'
        }
      }
    }
    stage('Load Image into Minikube') {
      steps {
        script {
          // Load the Docker image into Minikube
          sh "minikube -p ${MINIKUBE_PROFILE} image load ${DOCKER_IMAGE}"
        }
      }
    }
    stage('Deploying to Minikube') {
      steps {
        script {
          // Apply the Kubernetes deployment and service
          sh '/usr/local/bin/kubectl apply -f deployment.yaml -f service.yaml'
        }
      }
    }
  }
}
