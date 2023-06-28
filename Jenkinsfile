pipeline {
   tools {
        maven 'Maven'
    }
    agent {label 'slave-node-label'}
    environment {
        registry = "514337728862.dkr.ecr.ap-northeast-1.amazonaws.com/sudip-28june-repo"
    }
   
    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/aemdevopssudip/springboot-app.git']]])     
            }
        }
      stage ('Build') {
          steps {
            sh 'mvn clean install'           
            }
      }
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry 
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 514337728862.dkr.ecr.ap-northeast-1.amazonaws.com'
                sh 'docker push 514337728862.dkr.ecr.ap-northeast-1.amazonaws.com/sudip-28june-repo:latest'
         }
        }
      }

       stage('K8S Pre-deploy') {
        steps{   
            script {
                withKubeConfig([credentialsId: 'K8S', serverUrl: '']) {
                sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'
                sh 'chmod u+x ./kubectl'
                sh './kubectl delete -f eks-deploy-k8s.yaml'
                }
            }
        }
       }

      stage('K8S Deploy') {
        steps{   
            script {
                withKubeConfig([credentialsId: 'K8S', serverUrl: '']) {
                sh './kubectl apply -f eks-deploy-k8s.yaml'
                }
            }
        }
       } 
    }
}
