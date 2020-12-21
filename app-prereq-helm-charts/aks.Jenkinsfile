pipeline {
  environment {
    dockerRegistry = "${env.dockerRegistry}"
    dockerRegistryCredential = "${env.dockerRegistryCredential}"
    dockerImage = ''
  }
  agent any
  tools {nodejs "node" }
  stages {
    stage('Show GIT_URL') {
      steps {
        sh "echo ${env.GIT_URL}"
      }
    }
    stage('AZ Login') {
      steps {
        sh "az login --service-principal --username ${env.APP_ID}  --password ${env.PASSWORD} --tenant ${env.TENANT}"
      }
    }
    stage('Export KubeConfig Credentials ') {
      steps {
        //Exporting kubeConfig for kubectl so helm can upgrade
        sh "az aks get-credentials --name ${env.AKS_CLUSTER_NAME} --resource-group ${env.AKS_RESOURCE_GROUP} --overwrite-existing"
      }
    }
    stage('Show kubectl nodes') {
      steps {
        sh "echo showing_nodes"
        sh "kubectl get node"
        sh "kubectl get pod"
      }
    }
    stage('Cloning Git') {
      steps {
        git credentialsId: 'git_fork_private_key', url: "${env.GIT_URL}"
      }
    }
    stage('Get last git commit') {
       steps {
           script {
                git_hash = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
           }
       }
    }
    stage('Upgrade helm charts with helmfile') {
      steps{
        sh "helmfile -e gke sync"
        //sh "./linux-amd64/helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator --force-update"
        //sh "./linux-amd64/helm upgrade kafka incubator/kafka -f pre-req/values.yaml"
        //sh "./linux-amd64/helm upgrade kafka incubator/kafka -f pre-req/values.yaml --recreate-pods"
      }
    }
  }
}
