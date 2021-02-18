pipeline {
    agent any
  environment {
     SVC_ACCOUNT_KEY = credentials('terraform-auth')
     PROJECT_ID = 'stoked-genius-302113'
	 APP_NAME = 'tomcatapp'
     imageTag = 'gcr.io/$PROJECT_ID/$APP_NAME:v1.$BUILD_NUMBER'
     namespace = 'tomcat'
	 
  }
  stages {
    stage('Checkout SCM') {
      steps {
        checkout scm
        sh 'mkdir -p creds'
        sh 'echo $SVC_ACCOUNT_KEY | base64 -d > ./creds/demoaccount.json'
      }
    }
    stage('Install Docker') {
	  steps {
        script {
          demoapp = docker.build("$imageTag")
        }
      }
    }	
    stage('Build Image') {
	  steps {
        script {
          demoapp = docker.build("$imageTag")
        }
      }
    }
    stage("Push Image") {
	  steps {
        script {
          docker.withRegistry('https://gcr.io', 'gcr:gcr-credentials') {
            demoapp.push("$BUILD_NUMBER")
          }
        }
      }	  
    }
    stage('Application Deployment') {
      steps {
        sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"'  
        sh 'chmod u+x ./kubectl'
        sh 'export PATH=$PATH:$HOME'		
        sh "sshpass -p 'demo123' scp demo@k8s-master.us-central1-c.c.stoked-genius-302113.internal:/home/demo/.kube/config .kube/config"
        sh 'kubectl create ns $namespace'
        sh 'sed -i "s/tomcatapp:v1/tomcatapp:v1.$BUILD_NUMBER/g" tomcat.yaml'		
        sh 'kubectl --namespace=$namespace apply -f tomcat.yaml'
        //sh "echo http://`kubectl --namespace=$namespace get service/$feSvcName --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > $feSvcName"		
      }
    }
  }
}