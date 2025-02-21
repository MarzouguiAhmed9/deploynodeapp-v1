pipeline {
    agent any

    environment {
        registryCredential = 'dockerhublogin'  // Your Docker Hub credentials ID in Jenkins
        pushedImage = "ahmed20007/dockerimage"  // The correct Docker Hub image name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'  // Replace with your repo
            }
        }

        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build("dockerimage:latest")  // Build locally
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        sh "docker tag dockerimage:latest ${pushedImage}:latest"  // Correct tagging
                        sh "docker push ${pushedImage}:latest"  // Push with correct tag
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying ${pushedImage}:latest..."
                    // Add deployment steps (e.g., Kubernetes, Docker Compose, etc.)
                }
            }
        }
    }

    post {
        success {
            echo "Build and push Successful! ✅"
        }
        failure {
            echo "Pipeline failed! ❌"
        }
    }
}
