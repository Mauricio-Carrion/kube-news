pipeline{
  agent any
  //CI - integração continua
  stages {
    stage('Build Docker Image'){
      steps{
        script{
          dockerapp = docker.build("mauriciocarrion/kube-news:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
        }
      }
    }

    stage('Push Docker Image'){
      steps{
        script{
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub'){
            dockerapp.push('latest')
            dockerapp.push("${env.BUILD_ID}")
          }
        }
      }
    }

    stage('Deploy Kubernets'){
      steps{
        withCredentials([credentialsId: 'kubeconfig']){
          sh 'kubectl apply -f ./k8s/deployment.yaml'
        }
      }
    }
  }
}