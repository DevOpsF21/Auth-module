pipeline {
  agent any
  environment {
    // Define paths as environment variables for easy changes
    DOCKER_CMD = '/usr/local/bin/docker'
    KUBECTL_CMD = '/usr/local/bin/kubectl'
    MINIKUBE_CMD = '/opt/homebrew/bin/minikube'
  }
  stages {
    stage('Setup Minikube Docker Env') {
      steps {
        script {
          // Setup the Docker CLI to use Minikube's Docker daemon
          sh "eval \$(${env.MINIKUBE_CMD} docker-env)"
        }
      }
    }
    stage('Building Docker Image') {
      steps {
        script {
          // Building the Docker image using Minikube's Docker daemon
          sh "${env.DOCKER_CMD} build -t auth-module:latest ."
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
          // Triggering a rolling restart to ensure the latest image is used
          sh "${env.KUBECTL_CMD} rollout restart deployment/auth-module-deployment"
        }
      }
    }
  }
  post {
    success {
      echo 'Pipeline completed successfully.'
    }
    failure {
      echo 'Pipeline failed.'
      // Add more commands here for failure scenarios, like sending notifications.
    }
    always {
      // Cleanup action after every build
      echo 'Cleaning up resources...'
      // Consider removing unused Docker images or performing other cleanup tasks
      // sh "${env.DOCKER_CMD} rmi auth-module:latest || true"
    }
  }
}
