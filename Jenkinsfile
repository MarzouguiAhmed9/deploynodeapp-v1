pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
    }

    agent any

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/MarzouguiAhmed9/deploynodeapp-v1.git'
            }
        }

        stage('Build image') {
            steps {
                script {
                    // Build the Docker image
                    dockerImage = docker.build(dockerimagename)
                }
            }
        }

        stage('Pushing Image') {
            environment {
                registryCredential = 'dockerhublogin'
            }
            steps {
                script {
                    // Tag the image with your Docker Hub username and repository
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        // Tag the image with the appropriate Docker Hub repository
                        sh "docker tag ${dockerimagename} ${pushedImage}:latest"
                        // Push the image to Docker Hub with the correct tag
                        sh "docker push ${pushedImage}:latest"
                    }
                }
            }
        }

        stage('Deploying App to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
                }
            }
        }

    }
}
