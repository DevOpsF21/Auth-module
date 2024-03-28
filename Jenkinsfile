pipeline {
    agent any

    environment {
        // Environment variables
        DOCKER_IMAGE = "auth-module"
        DOCKER_TAG = "v1.0.${BUILD_NUMBER}"
        IMAGE_FULL_NAME = "${DOCKER_IMAGE}:${DOCKER_TAG}"
        DEPLOYMENT_NAME = "auth-module-deployment"
        CONTAINER_NAME = "auth-module"
        MINIKUBE_PATH = "/opt/homebrew/bin"
        DOCKER_PATH = "/usr/local/bin"
        POSTMAN_COLLECTION = "Authcollection.postman_collection.json"
    }

    stages {
        stage('Notify Deployment Start') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh "curl -X POST -H 'Content-type: application/json' -d '{\"title\": \"Deployment started\", \"text\": \"Deploying ${env.JOB_NAME} build ${env.BUILD_NUMBER} to Minikube\", \"priority\": \"normal\", \"tags\": [\"environment:minikube\", \"branch:${env.BRANCH_NAME}\"], \"alert_type\": \"info\"}' https://api.datadoghq.com/api/v1/events?api_key=${DATADOG_API_KEY}"
                    }
                }
            }
        }
        stage('Preparation') {
            steps {
                script {
                    // Set PATH to include Docker and Minikube
                    env.PATH = "${env.DOCKER_PATH}:${env.MINIKUBE_PATH}:${env.PATH}"
                }
            }
        }

        stage('Enable Ingress Addon') {
            steps {
                script {
                    // Check if Ingress is enabled and enable if not
                    def ingressEnabled = sh(script: "minikube addons list | grep ingress | grep enabled", returnStdout: true).trim()
                    if (ingressEnabled == '') {
                        sh "minikube addons enable ingress"
                    }
                }
            }
        }

        stage('Building Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_FULL_NAME} ."
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
                script {
                    // Use Minikube's Docker environment
                    sh 'eval $(minikube -p minikube docker-env)'
                    // Deploy application
                    sh "kubectl apply -f deployment.yaml -f service.yaml"
                    // Update deployment to use the built image
                    sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_FULL_NAME}"
                }
            }
        }

        stage('Apply Ingress Configuration') {
            steps {
                script {
                    // Apply Ingress resource
                    sh "kubectl apply -f ingress.yaml"
                }
            }
        }

        stage('Postman Testing') {
            steps {
                script {
                    try {
                        // Run tests via Postman collection
                        sh "newman run ${POSTMAN_COLLECTION}"
                    } catch (Exception e) {
                        echo "Postman tests failed but build continues..."
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Ensure deployment rollout is successful
                    sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
                    // Optionally verify running pods
                    sh "kubectl get pods --selector=app=${CONTAINER_NAME}"
                }
            }
        }
    }
}
