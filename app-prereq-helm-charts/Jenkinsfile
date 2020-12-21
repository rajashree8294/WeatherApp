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
    stage('Export kubecfg via kops') {
      steps {
        //Exporting kubecfg for kubectl so helm can upgrade
        sh 'kops export kubecfg'
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
        sh "helmfile sync"
        //sh "./linux-amd64/helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator --force-update"
        //sh "./linux-amd64/helm upgrade kafka incubator/kafka -f pre-req/values.yaml"
        //sh "./linux-amd64/helm upgrade kafka incubator/kafka -f pre-req/values.yaml --recreate-pods"
      }
    }
  }
}
