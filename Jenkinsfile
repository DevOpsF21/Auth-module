pipeline {
    agent any
    environment {
        // Define the Docker image you want to transfer and use, dynamically setting the version tag with each build
        DOCKER_IMAGE = "auth-module"
        DOCKER_TAG = "v1.0.${BUILD_NUMBER}" // Dynamically includes Jenkins build number
        IMAGE_FULL_NAME = "${DOCKER_IMAGE}:${DOCKER_TAG}"
        // Use the deployment name from your Kubernetes deployment manifest
        DEPLOYMENT_NAME = "auth-module-deployment"
        // Use the container name from your Kubernetes deployment manifest
        CONTAINER_NAME = "auth-module"
        // Paths to your Minikube and Docker binaries
        MINIKUBE_PATH = "/opt/homebrew/bin"
        DOCKER_PATH = "/usr/local/bin"
        // Path to Postman collection file in your Git repository (fixed redundancy)
        POSTMAN_COLLECTION = "${WORKSPACE}/Authcollection.postman_collection.json"
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
                    // Stop and remove the existing container if running
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    // Run the new container with the updated image on port 3000
                    sh "docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_FULL_NAME}"
                }
            }
        }
        stage('Transfer Image to Minikube') {
            steps {
                script {
                    // Save the Docker image to a tar file
                    sh "docker save ${IMAGE_FULL_NAME} > image.tar"
                    // Load the image into Minikube's Docker environment
                    sh "minikube -p minikube image load image.tar"
                    // Clean up the tar file after loading
                    sh "rm image.tar"
                }
            }
        }
        stage('Deploying to Minikube') {
            steps {
                script {
                    // Ensure kubectl is using Minikube's Docker environment
                    sh 'eval $(minikube -p minikube docker-env)'
                    
                    // Check if the deployment exists
                    def deploymentExists = sh(script: "kubectl get deployment ${DEPLOYMENT_NAME}", returnStatus: true) == 0
                    if (deploymentExists) {
                        // Update the deployment to use the new Docker image
                        sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_FULL_NAME}"
                        
                        // Restart the pods
                        sh "kubectl rollout restart deployment/${DEPLOYMENT_NAME}"
                    } else {
                        // Apply the deployment and service YAML files
                        sh "kubectl apply -f deployment.yaml -f service.yaml"
                    }
                }
            }
        }
        stage('Postman Testing') {
            steps {
                // Run Postman tests
                sh "newman run ${POSTMAN_COLLECTION}"
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    // Check the rollout status to ensure it's successful
                    sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
                    // Optionally, list the running pods to verify the update
                    sh "kubectl get pods --selector=app=${CONTAINER_NAME}"
                }
            }
        }
    }
}