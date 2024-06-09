pipeline {
    agent any

    stages {
        stage('Declarative: Checkout SCM') {
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
                sh 'docker build -t koyaadeniji/app-demo:${env.GIT_COMMIT} .'
            }
        }
        stage('Publish to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: 'https://index.docker.io/v1/']) {
                    sh 'docker push koyaadeniji/app-demo:${env.GIT_COMMIT}'
                }
            }
        }
        stage('Install kubectl') {
            steps {
                sh '''
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.29.0/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    mv ./kubectl /usr/local/bin/kubectl
                    kubectl version --client
                '''
            }
        }
        stage('Deploy to Cluster') {
            steps {
                sh 'aws eks update-kubeconfig --region us-east-1 --name ekscluster'
                sh 'kubectl apply -f deployment.yaml'
            }
        }
        stage('Confirm Deployment') {
            steps {
                script {
                    def retries = 5
                    def deploymentSucceeded = false

                    for (int i = 0; i < retries; i++) {
                        def pods = sh(script: "kubectl get pods --selector=app=app-demo -o jsonpath='{.items[*].status.phase}'", returnStdout: true).trim()
                        if (pods.contains("Running")) {
                            deploymentSucceeded = true
                            break
                        } else {
                            sleep(30)
                        }
                    }

                    if (!deploymentSucceeded) {
                        error("Deployment failed or pods are not running.")
                    } else {
                        echo "Deployment confirmed: Pods are running."
                    }
                }
            }
        }
    }
}

