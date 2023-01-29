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

    stage('Auth gcloud and download kubeconfig'){
      steps{
        withCredentials([file(credentialsId: 'gcpauth', variable: 'GCPAUTH')] ,){
          sh '''
            gcloud auth activate-service-account --key-file="$GCPAUTH"
            gcloud container clusters get-credentials k8s --region us-central1 --project jornada-376212
          '''
        }
      }
    }

    stage('Deploy Kubernets'){
      environment{
        tag_version = "${env.BUILD_ID}"
      }
      steps{
        withKubeConfig(credentialsId: 'kubeconfig'){
          sh 'sed -i "s/{{TAG}}/$tag_version/g" ./k8s/deployment.yaml'
          sh 'kubectl apply -f ./k8s/deployment.yaml'
        }
      }
    }
  }
}