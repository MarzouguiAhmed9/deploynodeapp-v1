pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
        dockerPAT = "dckr_pat_fyfqzqTZu0jRdTHxwZtPBLW7Gu0" // Docker Personal Access Token
        KUBE_CA_CERT_PATH = '/home/ahmed/.minikube/ca.crt'  // Path to the existing Kubernetes certificate
        KUBE_CREDENTIALS = 'kubernetes'  // Reference to the Jenkins Kubernetes credential ID
        KUBERNETES_URL = 'http://127.0.0.1:8081'  // Updated Minikube API URL
        SONAR_TOKEN = 'jenkins-sonar'  // SonarQube token for analysis
        SONAR_SCANNER_HOME = '/opt/sonar-scanner-4.7.0.2747-linux'  // Path to the sonar-scanner installation
        PATH = "$SONAR_SCANNER_HOME/bin:$PATH"  // Add sonar-scanner's bin directory to the PATH
    }

    agent any

    stages {
        stage('Fix Kubernetes Permissions') {
            steps {
                script {
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

        stage('SonarQube analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonar', installationName: 'sq1') {
                        // Run the SonarQube scan with SonarScanner for Node.js (ensure this script runs in the root folder of your Node.js app)
                        sh 'sonar-scanner -Dsonar.projectKey=nodeapp -Dsonar.login=$SONAR_TOKEN'
                    }
                }
            }
        }

        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build(dockerimagename)  // Build Docker image using the Dockerfile for Node.js
                }
            }
        }

        stage('Pushing Image') {
            environment {
                registryCredential = 'dockerhublogin'
            }
            steps {
                script {
                    sh "echo $dockerPAT | docker login -u ahmed20007 --password-stdin"
                    sh "docker tag ${dockerimagename} ${pushedImage}:latest"
                    sh "docker push ${pushedImage}:latest"
                }
            }
        }

        stage('Deploying App to Kubernetes') {
            steps {
                script {
                    sh "sudo chmod 644 ${KUBE_CA_CERT_PATH}"
                    sh "sudo chown ahmed:ahmed ${KUBE_CA_CERT_PATH}"

                    withEnv(["KUBERNETES_URL=${KUBERNETES_URL}"]) {
                        kubernetesDeploy(
                            configs: "deploymentservice.yml",  // Update this to the correct YAML file for your Node.js deployment
                            kubeconfigId: KUBE_CREDENTIALS
                        )
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up Kubernetes certificate..."
            // Uncomment if you need to remove the certificate file after use
            // sh "rm -f ${KUBE_CA_CERT_PATH}"
        }
    }
}
