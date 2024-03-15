pipeline {
  agent any

  environment {
    // Define the Docker image you want to transfer and use
    DOCKER_IMAGE = "auth-module:latest"
    // Use the deployment name from your Kubernetes deployment manifest
    DEPLOYMENT_NAME = "auth-module-deployment"
    // Use the container name from your Kubernetes deployment manifest
    CONTAINER_NAME = "auth-module"
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

    stage('Run Docker Container Locally') {
      steps {
        sh '''
          # Stop and remove the existing container if it exists
          docker stop auth-container || true
          docker rm auth-container || true
          
          # Run a new container from the built image
          docker run -d --name auth-container -p 3000:3000 ${DOCKER_IMAGE}
        '''
      }
    }

    stage('Transfer Image to Minikube') {
      steps {
        sh '''
          # Save the Docker image to a tar file
          docker save ${DOCKER_IMAGE} > image.tar
          
          # Load the image into Minikube's Docker environment
          minikube -p minikube image load image.tar
          
          # Clean up the tar file after loading
          rm image.tar
        '''
      }
    }

    stage('Deploying to Minikube') {
    steps {
        sh '''
            # Ensure kubectl is using Minikube's Docker environment
            eval $(minikube -p minikube docker-env)
            
            # Update the deployment to use the new Docker image
            kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${DOCKER_IMAGE}
            
            # Trigger a rollout restart of the deployment to refresh the pods
            kubectl rollout restart deployment/${DEPLOYMENT_NAME}
        '''
    }
   }
  }
}
