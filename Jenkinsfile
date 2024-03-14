pipeline {
    agent any
    environment {
        // Minikube Docker environment variables
        DOCKER_TLS_VERIFY = "1"
        DOCKER_HOST = "tcp://127.0.0.1:60076"
        DOCKER_CERT_PATH = "/Users/khuloodi/.minikube/certs"
        MINIKUBE_ACTIVE_DOCKERD = "minikube"
        // Docker command path
        DOCKER_CMD = '/usr/local/bin/docker'
        // kubectl command path (assuming it's installed and configured on Jenkins agent)
        KUBECTL_CMD = '/usr/local/bin/kubectl'
    }
    stages {
        stage('Setup Minikube Docker Environment') {
            steps {
                script {
                    // No physical commands here, just a placeholder to remind us we're using Minikube's Docker
                    echo "Assuming Minikube's Docker environment variables are correctly set for the whole pipeline."
                }
            }
        }
        stage('Building Docker Image') {
            steps {
                script {
                    sh """
                    export DOCKER_TLS_VERIFY=\${env.DOCKER_TLS_VERIFY}
                    export DOCKER_HOST=\${env.DOCKER_HOST}
                    export DOCKER_CERT_PATH=\${env.DOCKER_CERT_PATH}
                    export MINIKUBE_ACTIVE_DOCKERD=\${env.MINIKUBE_ACTIVE_DOCKERD}
                    \${env.DOCKER_CMD} build -t auth-module:latest .
                    """
                }
            }
        }
        stage('Deploying to Minikube') {
            steps {
                script {
                    sh """
                    export DOCKER_TLS_VERIFY=\${env.DOCKER_TLS_VERIFY}
                    export DOCKER_HOST=\${env.DOCKER_HOST}
                    export DOCKER_CERT_PATH=\${env.DOCKER_CERT_PATH}
                    export MINIKUBE_ACTIVE_DOCKERD=\${env.MINIKUBE_ACTIVE_DOCKERD}
                    \${env.KUBECTL_CMD} apply -f deployment.yaml -f service.yaml
                    \${env.KUBECTL_CMD} rollout restart deployment/auth-module-deployment
                    """
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
            // Cleanup or final actions that should always run
            echo 'Pipeline execution is complete.'
        }
    }
}
