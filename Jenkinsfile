pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "amu2112/k8s-platform"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Amu2112/k8s-platform.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl get nodes
                    kubectl apply -f overlays/prod/
                    '''
                }
            }
        }
    }
}
