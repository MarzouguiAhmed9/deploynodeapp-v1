pipeline {
    agent any

    environment {
        registryCredential = 'dockerhublogin' // Jenkins credentials for Docker Hub login
        dockerImageName = 'nodeapp' // The name of the Docker image
        dockerHubUsername = 'ahmed20007' // Your Docker Hub username
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    // Build the Docker image
                    dockerImage = docker.build("${dockerImageName}:v2")
                }
            }
        }

        stage('Tag Image') {
            steps {
                script {
                    // Tag the image with your Docker Hub username and repository
                    dockerImage.tag("${dockerHubUsername}/${dockerImageName}:v2")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    // Push the image to Docker Hub with the specified tag
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push("${dockerHubUsername}/${dockerImageName}:v2")
                    }
                }
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
