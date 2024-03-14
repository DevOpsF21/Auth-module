pipeline {
  agent any

  environment {
    // Assuming "/path/to/docker/bin" is where Docker is installed. Update this path as necessary.
    DOCKER_PATH = "/usr/local/bin"
    MINIKUBE_PATH = "/opt/homebrew/bin"
  }

  stages {
    stage('Preparation') {
      steps {
        script {
          // This is a better way to dynamically adjust the PATH for all the following stages.
          env.PATH = "${env.DOCKER_PATH}:${env.MINIKUBE_PATH}:${env.PATH}"
        }
      }
    }

    stage('Building Docker Image') {
      steps {
        sh 'docker build -t auth-module:latest .'
      }
    }

    stage('Run Docker Container Locally') {
      steps {
        sh '''
          docker stop auth-container || true
          docker rm auth-container || true
          docker run -d --name auth-container -p 3000:3000 auth-module:latest
        '''
      }
    }

    stage('Deploying to Minikube') {
      steps {
        sh '''
          # Debugging: Print the current PATH to ensure our adjustments are effective
          echo "Current PATH: $PATH"
          
          # Verify Docker is correctly installed and accessible
          which docker || echo "Docker not found"
          
          # Configure shell to use Minikube's Docker daemon
          eval $(minikube -p minikube docker-env -u)
          
          # Insert your deployment commands here
          # For example, applying Kubernetes configurations
          kubectl apply -f deployment.yaml -f service.yaml
        '''
      }
    }
  }
}
