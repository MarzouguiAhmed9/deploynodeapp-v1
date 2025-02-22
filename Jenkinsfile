pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
        dockerPAT = "dckr_pat_fyfqzqTZu0jRdTHxwZtPBLW7Gu0" // Docker Personal Access Token
        KUBE_CA_CERT_PATH = '/home/ahmed/.minikube/ca.crt'  // Path to the existing Kubernetes certificate
        KUBE_CREDENTIALS = 'kubernetes'  // Reference to the Jenkins Kubernetes credential ID
        KUBERNETES_URL = 'http://127.0.0.1:8081'  // Updated Minikube API URLl
                SONARQUBE_SERVER = 'jenkins-sonar'  // Set in Jenkins > Manage Jenkins > Configure System

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
           stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('jenkins-sonar')  // Using your SonarQube token ID
            }
            steps {
                script {
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh """
                            sonar-scanner \
                            -Dsonar.projectKey=nodeapp \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://127.0.0.1:9000 \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.qualitygate.wait=true
                        """
                    }
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
                    // Build the Docker imagee
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

                    // Set the Kubernetes API URL for Minikube (using the KUBERNETES_URL environment variable)
                    withEnv(["KUBERNETES_URL=${KUBERNETES_URL}"]) {
                        // Deploy the app to Kubernetes using the Jenkins Kubernetes credentials
                        kubernetesDeploy(
                            configs: "deploymentservice.yml",  // Path to your Kubernetes YAML file
                            kubeconfigId: KUBE_CREDENTIALS,    // Reference to the Kubernetes credentials in Jenkins
                        )
                    }
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
