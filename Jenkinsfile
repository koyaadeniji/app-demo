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
                script {
                    def imageTag = "${env.GIT_COMMIT}"
                    def imageName = "koyaadeniji/app-demo:${imageTag}"
                    sh """
                        echo "Building Docker image: ${imageName}"
                        docker build -t ${imageName} .
                    """
                }
            }
        }
        stage('Publish to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: 'https://index.docker.io/v1/']) {
                    script {
                        def imageTag = "${env.GIT_COMMIT}"
                        def imageName = "koyaadeniji/app-demo:${imageTag}"
                        sh """
                            echo "Pushing Docker image: ${imageName}"
                            docker push ${imageName}
                        """
                    }
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
                sh '''
                    echo "Updating kubeconfig"
                    aws eks update-kubeconfig --region us-east-1 --name ekscluster
                    echo "Applying Kubernetes deployment"
                    kubectl apply -f k8s/deployment.yaml
                '''
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

