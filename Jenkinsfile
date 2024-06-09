pipeline {
    agent any

    environment {
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
                sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl'
                sh 'chmod +x ./kubectl'
                sh './kubectl version --client'
            }
        }

        stage('Deploy to Cluster') {
            steps {
                sh 'aws eks update-kubeconfig --region us-east-1 --name ekscluster'
                sh 'envsubst < ${WORKSPACE}/deployment.yaml | ./kubectl apply -f -'
            }
        }

        stage('Delete Images') {
            steps {
                sh 'docker rmi -f $(docker images -q -a)'
            }
        }
    }

    post {
        failure {
            mail to: 'your-email@example.com',
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}

