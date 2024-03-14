pipeline {
  agent any

  stage('Preparation') {
  steps {
    sh 'export PATH=$PATH:/path/to/docker'
  }
}

  stages {
    stage('Building Docker Image') {
      steps {
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
    sh 'echo $PATH' // Print current PATH for debugging
    sh '/usr/bin/which docker || echo "Docker not found"' // Check if Docker can be found
    sh '''
    # Adjust PATH if necessary
    export PATH=$PATH:/path/to/docker/bin
    
    # Attempt to set Minikube's Docker environment
    eval $('/opt/homebrew/bin/minikube' -p minikube docker-env)
    
    # Your deployment commands here
    '''
  }
}

  }
}
