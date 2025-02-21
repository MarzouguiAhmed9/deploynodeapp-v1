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
        registryCredential = 'dockerhublogin'
      }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
           sh "docker tag dockerimage:latest ${pushedImage}:latest"  // Correct tagging
                        sh "docker push ${pushedImage}:latest"  // Push with correct tag          }
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
