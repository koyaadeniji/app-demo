pipeline {
    agent any

    environment {
        REPOSITORY_TAG = "${PRIVATE_REPO_TAG}/${PRIVATE_APP_NAME}:${VERSION}"
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
    }

    post {
        failure {
            mail to: 'your-email@example.com',
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}

