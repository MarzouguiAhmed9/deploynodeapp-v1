pipeline {
    environment {
        dockerimagename = "nodeapp:v2"
        dockerImage = ""
        pushedImage = "ahmed20007/nodeapp"
        dockerPAT = credentials('dockerhub-token')  // Securely store Docker PAT in Jenkins credentials
        KUBE_CA_CERT_PATH = '/home/ahmed/.minikube/ca.crt'
        KUBE_CREDENTIALS = 'kubernetes'
        KUBERNETES_URL = 'http://127.0.0.1:8081'
        SONARQUBE_SERVER = 'jenkins-sonar'  // Set in Jenkins > Manage Jenkins > Configure System
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

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build(dockerimagename)
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    sh "echo ${dockerPAT} | docker login -u ahmed20007 --password-stdin"
                    sh "docker tag ${dockerimagename} ${pushedImage}:latest"
                    sh "docker push ${pushedImage}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
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
            echo "Pipeline execution completed."
        }
    }
}
