pipeline {
    agent any

    environment {
        REPOSITORY_TAG = "${REPO_TAG}/${APP_NAME}:${VERSION}"
        REPO_TAG = "koyaadeniji"
        APP_NAME = "app-demo"
        VERSION = "${BUILD_ID}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t koyaadeniji/app-demo:${GIT_COMMIT} .'
            }
        }

        stage('Publish to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                    sh 'docker push koyaadeniji/app-demo:${GIT_COMMIT}'
                }
            }
        }

        stage('Install kubectl') {
            steps {
                sh 'curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt'
		sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.29/bin/linux/amd64/kubectl'
                sh 'chmod +x ./kubectl'
                sh 'mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH'
            }
        }

        stage('Deploy to EKS Cluster') {
            steps {
                sh 'aws eks update-kubeconfig --region us-east-1 --name ekscluster'
                sh 'envsubst < deployment.yml | kubectl apply -f -'
            }
        }

        stage('Clean up') {
            steps {
                sh 'docker rmi -f $(docker images -q koyaadeniji/app-demo)'
            }
        }
    }
}

