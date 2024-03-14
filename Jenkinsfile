pipeline {
  agent any

  environment {
    // Define the Docker image you want to transfer
    DOCKER_IMAGE = "auth-module:latest"
    // Paths to your Minikube and Docker binaries if necessary
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

    stage('Transfer Image to Minikube') {
      steps {
        sh '''
          # Save the Docker image to a tar file
          docker save ${DOCKER_IMAGE} > image.tar
          
          # Set shell to use Minikube's Docker environment
          #eval $(minikube -p minikube docker-env)
          
          # Load the image from the tar file into Minikube's Docker environment
          docker load < image.tar
          
          # Clean up the tar file
          rm image.tar
        '''
      }
    }

    stage('Deploying to Minikube') {
      steps {
        sh '''
          # Deployment commands don't need to change, since the image is now available in Minikube
          kubectl apply -f deployment.yaml -f service.yaml
        '''
      }
    }
  }
}
