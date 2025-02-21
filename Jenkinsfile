pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
        dockerPAT = "dckr_pat_fyfqzqTZu0jRdTHxwZtPBLW7Gu0" // Docker Personal Access Token
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
                    // Login using the Docker Personal Access Token (PAT)
                    sh "echo $dockerPAT | docker login -u ahmed20007 --password-stdin"

                    // Tag the image with your Docker Hub username and repository
                    sh "docker tag ${dockerimagename} ${pushedImage}:latest"
                    
                    // Push the image to Docker Hub with the correct tag
                    sh "docker push ${pushedImage}:latest"
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
