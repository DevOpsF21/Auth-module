pipeline {
  agent any
  stages {
    stage('Building Docker Image') {
      steps {
        sh '/usr/local/bin/docker build -t auth-module:latest .'
      }
    }
    stage('Run Docker Container Locally') {
      steps {
        sh '''
       #/usr/local/bin/docker stop auth-container
       #/usr/local/bin/docker rm auth-container
        /usr/local/bin/docker run -d --name auth-container -p 3000:3000 auth-module:latest
        '''
      }
    }
    stage('Deploying to Minikube') {
      steps {
        sh '''
        # Configuring shell to use Minikube's Docker daemon
        /tmp/eval $(/opt/homebrew/bin/minikube -p minikube docker-env)
        # Now, any Docker commands will interact with Minikube's Docker daemon

        # Deploying using kubectl within the same shell to ensure Docker env vars are still set
        /usr/local/bin/kubectl apply -f deployment.yaml -f service.yaml

        # Optionally, if you want to revert to using the system's Docker daemon within this block for subsequent commands
        # eval $(/usr/local/bin/minikube -p minikube docker-env -u)
        '''
      }
    }
  }
}
