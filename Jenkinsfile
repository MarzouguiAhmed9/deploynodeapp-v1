pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
        dockerPAT = "dckr_pat_fyfqzqTZu0jRdTHxwZtPBLW7Gu0" // Docker Personal Access Token
        KUBE_CA_CERT_PATH = '/home/ahmed/.minikube/ca.crt'  // Path to the existing Kubernetes certificate
        KUBE_CREDENTIALS = 'kubernetes'  // Reference to the Jenkins Kubernetes credential ID
        KUBERNETES_URL = 'http://127.0.0.1:8081'  // Updated Minikube API URL
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

        stage('Scan') {
            steps {
                withSonarQubeEnv('sq1') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=nodeapp \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://127.0.0.1:9000 \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.qualitygate.wait=true
                    '''
                }
            }
        }

        stage('Build image') {
            steps {
                script {
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
                            configs: "deploymentservice.yml",
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
