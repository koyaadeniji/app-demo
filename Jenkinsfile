pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "koyaadeniji/app-demo:${BUILD_ID}"
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        KUBECONFIG_CREDENTIAL_ID = 'your-kubeconfig-credential-id'
        K8S_NAMESPACE = 'interswitch'
        K8S_DEPLOYMENT_NAME = 'spring-boot-deployment'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/koyaadeniji/app-demo.git'
            }
        }

        stage('Check Prerequisites') {
            steps {
                script {
                    def dockerInstalled = sh(script: 'docker --version', returnStatus: true) == 0
                    def kubectlInstalled = sh(script: 'kubectl version --client', returnStatus: true) == 0

                    if (!dockerInstalled) {
                        error 'Docker is not installed on the agent.'
                    }
                    if (!kubectlInstalled) {
                        error 'Kubectl is not installed on the agent.'
                    }
                }
            }
        }

        stage('Remove Previous Deployment') {
            steps {
                script {
                    withKubeConfig([credentialsId: KUBECONFIG_CREDENTIAL_ID]) {
                        sh "kubectl -n ${K8S_NAMESPACE} delete deployment ${K8S_DEPLOYMENT_NAME} || true"
                        sh "kubectl -n ${K8S_NAMESPACE} delete svc spring-boot-service || true"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Dockerize') {
            steps {
                script {
                    def image = docker.build("${DOCKER_IMAGE}")
                    image.tag('latest')
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY, 'docker-hub-credentials-id') {
                        def image = docker.image("${DOCKER_IMAGE}")
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: KUBECONFIG_CREDENTIAL_ID]) {
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
