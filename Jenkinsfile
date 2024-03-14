pipeline {
  agent any

  environment {
    // Define the Docker image you want to pull and use
    DOCKER_IMAGE = "auth-module:latest"
    // Define the paths to your Minikube and Docker binaries if necessary
    MINIKUBE_PATH = "/opt/homebrew/bin"
    DOCKER_PATH = "/usr/local/bin"
  }

  stages {
    stage('Preparation') {
      steps {
        script {
          // Adjust the PATH to ensure Docker and Minikube commands are accessible
          env.PATH = "${env.DOCKER_PATH}:${env.MINIKUBE_PATH}:${env.PATH}"
        }
      }
    }

    stage('Building Docker Image') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE} .'
      }
    }

    stage('Run Docker Container Locally') {
      steps {
        sh '''
          docker stop auth-container || true
          docker rm auth-container || true
          docker run -d --name auth-container -p 3000:3000 ${DOCKER_IMAGE}
        '''
      }
    }

    stage('Pulling and Deploying to Minikube') {
      steps {
        sh '''
          # Configure shell to use Minikube's Docker daemon
          eval $(minikube -p minikube docker-env)
          
          # Pull the Docker image into Minikube's Docker environment
          docker pull ${DOCKER_IMAGE}
          
          # Apply Kubernetes configurations
          kubectl apply -f deployment.yaml -f service.yaml
        '''
      }
    }
  }
}
