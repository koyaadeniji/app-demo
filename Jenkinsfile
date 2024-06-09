pipeline {
  
  agent any

  environment {
    REPOSITORY_TAG     = "${PRIVATE_REPO_TAG}/${PRIVATE_APP_NAME}:${VERSION}"
    REPO_TAG           = "koyaadeniji"
    APP_NAME   	       = "app-demo"
    VERSION            = "${BUILD_ID}"
  }

  stages {
    stage ('Build') {
      steps {
        sh 'printenv'
        sh 'docker build -t koyaadeniji/app-demo:""$GIT_COMMIT"" .'
      }
    }
    
     stage ('Publish to DockerHub') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'docker push koyaadeniji/app-demo:""$GIT_COMMIT""'
         }
       }
     }


    stage ("Install kubectl"){
            steps {
                sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl'
                sh 'chmod +x ./kubectl'
                sh './kubectl version --client'
            }
        }
        
    stage ('Deploy to Cluster') {
            steps {
             	sh 'aws eks update-kubeconfig --region us-east-1 --name ekscluster'
                sh 'envsubst < ${WORKSPACE}/deployment.yaml | ./kubectl apply -f -'
            }
    }

    stage ('Delete Images') {
      steps {
        sh 'docker rmi -f $(docker images -q -a)'
      }
    }
   }
 }
