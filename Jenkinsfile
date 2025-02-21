pipeline {

  environment {
    dockerimagename = "nodeapp:v2"
    dockerImage = ""
  }

  agent any

  stages {

   stage('Checkout Source') {
     steps {
       git branch: 'main', url: 'https://github.com/MarzouguiAhmed9/deploynodeapp-v1.git'
     }
   }


    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build(dockerimagename)
        }
      }
    }

    stage('Pushing Image') {
      environment {
        registryCredential = 'dockerhublogin' // Jenkins credentials ID
      }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
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
