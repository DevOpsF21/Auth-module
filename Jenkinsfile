pipeline {
  agent any
  stages {
    stage('Building Docker Image') {
      steps {
        sh '/usr/share/man/man1/builtin.1/eval $(minikube docker-env)'
        sh '/usr/local/bin/docker build -t auth-module:latest .'
      }
    }
    stage('Run Docker Container Locally') {
      steps {
        sh '''
        /usr/local/bin/docker stop auth-container
        /usr/local/bin/docker rm auth-container
        /usr/local/bin/docker run -d --name auth-container -p 3000:3000 auth-module:latest
        '''
      }
    }
    stage('Deploying to Minikube') {
      steps {
        sh '/usr/local/bin/kubectl apply -f deployment.yaml -f service.yaml'
      }
    }
  }
}