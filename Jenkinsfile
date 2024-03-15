pipeline {
    agent any

    environment {
        // Define the Docker image you want to transfer and use, with a version tag placeholder
        DOCKER_IMAGE = "auth-module"
        DOCKER_TAG = "v1.0.0" // Consider dynamically setting this for each build
        IMAGE_FULL_NAME = "${DOCKER_IMAGE}:${DOCKER_TAG}"
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
                sh "docker build -t ${IMAGE_FULL_NAME} ."
            }
        }

      stage('Run Docker Container Locally') {
    steps {
        script {
            // Define the command as a variable
            def checkCommand = "docker ps -q -f name=^${CONTAINER_NAME}$"
            def isRunning = sh(script: checkCommand, returnStdout: true).trim()
            if (isRunning) {
                // Stop and remove the container if it is running
                sh "docker stop ${CONTAINER_NAME}"
                sh "docker rm ${CONTAINER_NAME}"
            }
            // Run the new container
            sh "docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_FULL_NAME}"
        }
    }
}

}

}


        stage('Transfer Image to Minikube') {
            steps {
                sh '''
                    # Save the Docker image to a tar file
                    docker save ${IMAGE_FULL_NAME} > image.tar

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

                    # Update the deployment to use the new Docker image and restart the pods
                    kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_FULL_NAME} --record
                    kubectl rollout restart deployment/${DEPLOYMENT_NAME}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Check the rollout status to ensure it's successful
                    sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
                    // Optionally, list the running pods
                    sh "kubectl get pods --selector=app=${CONTAINER_NAME}"
                }
            }
        }
    }
}
