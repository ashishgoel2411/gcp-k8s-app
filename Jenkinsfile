pipeline {
    agent any
  environment {
     SVC_ACCOUNT_KEY = credentials('terraform-auth')
     PROJECT_ID = 'stoked-genius-302113'
     APP_NAME = 'tomcatapp'
     imageTag  = 'gcr.io/$PROJECT_ID/$APP_NAME:v1.$BUILD_NUMBER'
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
    stage('Build Image') {
      steps {
        sh 'echo $PROJECT_ID $APP_NAME'	  
        sh 'env'
        sh 'docker image build -t $imageTag .'
        //sh 'docker image tag $APP_NAME:v1.$BUILD_NUMBER $imageTag/$APP_NAME:v1.$BUILD_NUMBER'
        //sh 'docker image tag $APP_NAME:v1.$BUILD_NUMBER $imageTag/$APP_NAME:latest'
      }
    }
    stage("Push Image") {
      steps {
        sh 'gcloud docker -- push $imageTag'
        //sh 'gcloud docker -- push $imageTag/$APP_NAME:latest'		 
      }
    }
    stage('Application Deployment') {
      steps {
        sh 'kubectl create ns $namespace'
        sh 'sed -i 's/gcr.io\/$PROJECT_ID\/$APP_NAME:v1/\$imageTag' ./*.yaml'
        sh 'kubectl --namespace=$namespace apply -f tomcat.yaml'
        sh 'echo http://`kubectl --namespace=$namespace get service/$feSvcName --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > $feSvcName'		
      }
    }
  }
}  