
pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'perezi3/ci-lab3'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/miamioh-cit/225-lab3-1.git'
        KUBECONFIG = credentials('perezi3-225')  // matches your Rancher kubeconfig secret file
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    bat "docker login -u %DOCKERHUB_USERNAME% -p %DOCKERHUB_PASSWORD%"
                    bat "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Dev Environment using NodePort') {
            steps {
                withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                    bat "powershell -Command \"(Get-Content deployment.yaml) -replace '${DOCKER_IMAGE}:latest', '${DOCKER_IMAGE}:${IMAGE_TAG}' | Set-Content deployment.yaml\""
                    bat "kubectl apply -f deployment.yaml"
                }
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                    bat "kubectl get all"
                }
            }
        }
    }
}
