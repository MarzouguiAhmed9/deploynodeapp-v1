pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"  // Define pushed image with your Docker Hub username
        KUBE_CA_CERT_PATH = '/home/ahmed/.minikube/ca.crt'  // Path to Kubernetes certificate
        KUBE_CREDENTIALS = 'kubernetes'  // Jenkins credential ID for Kubernetes
        KUBERNETES_URL = 'http://127.0.0.1:8081'  // Minikube API URL
        SONARQUBE_SERVER = 'jenkins-sonar'  // Jenkins SonarQube server ID
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

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('jenkins-sonar')  // Secure SonarQube token
            }
            steps {
                script {
                    withSonarQubeEnv(SONARQUBE_SERVER) {
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
        }

        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/MarzouguiAhmed9/deploynodeapp-v1.git'
            }
        }

        stage('Build Image') {
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
                    withCredentials([string(credentialsId: 'dockerhublogin', variable: 'DOCKER_PAT')]) {
                        sh '''
                            echo $DOCKER_PAT | docker login -u ahmed20007 --password-stdin
                            docker tag nodeapp:v2 ahmed20007/nodeapp:latest
                            docker push ahmed20007/nodeapp:latest
                        '''
                    }
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
        }
    }
}
