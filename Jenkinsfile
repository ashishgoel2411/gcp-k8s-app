pipeline {
  agent any
    environment {
      SVC_ACCOUNT_KEY = credentials('terraform-auth')
      PROJECT_ID = 'chaos-engineering-308202'
	  APP_NAME = 'tomcatapp'
      imageTag = 'gcr.io/$PROJECT_ID/$APP_NAME:v1.$BUILD_NUMBER'
      TOMCAT_NS = 'tomcat-ns'
      POSTGRES_NS = 'postgres-ns'
      RABBITMQ_NS = 'rabbitmq-ns'	  
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
            demoapp.push("v1.$BUILD_NUMBER")
          }
        }
      }	  
    }
    stage('Setup K8s access at Jenkins') {
      steps {
        sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"'  
        sh 'chmod u+x ./kubectl'
        sh 'export PATH=$PATH:$HOME'
        sh 'rm -rf $HOME/.kube'
        sh 'rm -rf $HOME/.ssh'
        sh 'mkdir $HOME/.kube'
        sh 'mkdir $HOME/.ssh'		
        sh 'echo > $HOME/.ssh/known_hosts'		
        sh "sshpass -p 'demo123' scp -r -o StrictHostKeyChecking=no demo@k8s-master.us-central1-c.c.chaos-engineering-308202.internal:/home/demo/.kube/* $HOME/.kube/"
       }
    }
    stage('Metrics Server Deployment') {
      steps {
        sh 'kubectl apply -f metrics-server.yaml'
      }
    }
    stage('Taint master for NoScheduled') {
      steps {
        sh 'kubectl taint nodes k8s-master node-role.kubernetes.io/k8s-master:NoSchedule'
      }
    }	
    stage('Tomcat Deployment') {
      steps {
        //sh 'rm $HOME/workspace/application-deployment/tomcat-*.yaml 2> /dev/null'
        sh 'kubectl create ns $TOMCAT_NS'
        sh 'cp tomcat.yaml tomcat-$BUILD_NUMBER.yaml'
        sh 'sed -i "s/tomcatapp:v1/tomcatapp:v1.$BUILD_NUMBER/g" tomcat-$BUILD_NUMBER.yaml'
        sh 'kubectl create secret docker-registry gcr-json-key --docker-server=gcr.io --docker-username=_json_key --docker-password="$(cat ./creds/demoaccount.json)" --docker-email=demoaccount@chaos-engineering-308202.iam.gserviceaccount.com -n $TOMCAT_NS'
        sh 'kubectl --namespace=$TOMCAT_NS apply -f tomcat-$BUILD_NUMBER.yaml'
        sh 'sleep 10'
      }
    }
    stage('Postgres Deployment') {
      steps {
        sh 'kubectl create ns $POSTGRES_NS'
        sh 'kubectl --namespace=$POSTGRES_NS apply -f postgres.yaml'
        sh 'sleep 10'			
      }
    }
    stage('RabbitMQ Deployment') {
      steps {
        sh 'kubectl create ns $RABBITMQ_NS'
        //sh 'kubectl run etcd --image=microbox/etcd --port=4001 --namespace=$RABBITMQ_NS -- --name etcd'
        //sh 'kubectl --namespace=$RABBITMQ_NS expose deployment etcd'
        //sh 'kubectl create secret generic --namespace=$RABBITMQ_NS erlang.cookie --from-file=./erlang.cookie'		
        sh 'kubectl --namespace=$RABBITMQ_NS apply -f rabbitmq.yaml'
        sh 'sleep 10'			
      }
    }	
  }
}