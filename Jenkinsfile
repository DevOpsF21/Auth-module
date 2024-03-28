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

        stage('Preparation') {
            steps {
                script {
                    // Set PATH to include Docker and Minikube
                    env.PATH = "${env.DOCKER_PATH}:${env.MINIKUBE_PATH}:${env.PATH}"
                }
            }
        }
        stage('Testing: find syntax errors using eslint') {
            steps {
                script {
                    try {
                        // Run ESLint to lint your JavaScript code
                        sh 'npx eslint .'
                    } catch (err) {
                        // Handle ESLint errors (e.g., echo error message)
                        echo "ESLint found errors but pipeline will continue: ${err}"
                    }
                }
            }
        }

        stage('Building Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_FULL_NAME} ."
            }
        }
        stage('Testing: Check Docker Image for vulnerability') {
            steps {
                snykSecurity(
                    snykInstallation: 'Snyk_security',
                    snykTokenId: 'Snyk_api_token',
                    failOnError: false,
                    failOnIssues: false,
                    additionalArguments: "--severity-threshold=high"
                )
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

        stage('Testing: Kubernetes Security Scan') {
            steps {
                snykSecurity(
                    snykInstallation: 'Snyk_security',
                    snykTokenId: 'Snyk_api_token',
                    failOnError: false,
                    failOnIssues: false,
                    additionalArguments: "--all-projects --severity-threshold=high"
                )
            }
        }
    }
}
