pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
        dockerPAT = "dckr_pat_fyfqzqTZu0jRdTHxwZtPBLW7Gu0" // Docker Personal Access Token
        KUBE_CA_CERT_PATH = '/home/ahmed/.minikube/ca.crt'  // Path to the existing Kubernetes certificate
        KUBE_CONFIG_PATH = '/home/ahmed/.kube/config'        // Path to the Kubernetes config
    }

    agent any

    stages {
        stage('Fix Kubernetes Permissions') {
            steps {
                script {
                    // Ensure the Kubernetes certificate file has correct permissions
                    sh 'sudo chmod 644 /home/ahmed/.minikube/ca.crt'
                    sh 'sudo chown ahmed:ahmed /home/ahmed/.minikube/ca.crt'
                }
            }
        }

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
                    // Ensure the certificate file has the correct permissions
                    sh "sudo chmod 644 ${KUBE_CA_CERT_PATH}"
                    sh "sudo chown ahmed:ahmed ${KUBE_CA_CERT_PATH}"

                    // Deploy the app to Kubernetes using the existing certificate and kubeconfig
                    kubernetesDeploy(
                        configs: "deploymentservice.yml",      // Path to your Kubernetes YAML file
                        kubeconfig: KUBE_CONFIG_PATH,          // Path to the kubeconfig file
                        kubeContext: "minikube",               // Context in your kubeconfig
                        kubeNamespace: "default",              // Namespace where you want to deploy
                        certificate: KUBE_CA_CERT_PATH        // Path to your certificate
                    )
                }
            }
        }
    }

    post {
        always {
            // Clean up the certificate file after use if necessary
            echo "Cleaning up Kubernetes certificate..."
            // You can choose to remove the certificate after use if it's no longer needed
            // sh "rm -f ${KUBE_CA_CERT_PATH}"
        }
    }
}
