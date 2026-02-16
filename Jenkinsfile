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

        stage('Debug Sonar Config') {
    steps {
        echo "Check SonarQube name in Jenkins Global Configuration"
    }
}

        stage('SAST - sonarsube') {
    steps {
        script {
            def scannerHome = tool 'SonarScanner'
        withSonarQubeEnv('sonarqube') {
             sh  """
               ${scannerHome}/bin/sonar-scanner \
               -Dsonar.projectKey=k8s-platform \
               -Dsonar.sources=.
                """
        }
    }
}    
}            

        stage('SonarQube Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}

        // ===== Dev Deployment =====
        stage('Deploy to Dev') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
                    kubectl apply -k overlays/dev/ -n dev
                    kubectl rollout status deployment/myapp -n dev --timeout=120s
                    '''
                }
            }
        }

        stage('Dev Health Check') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBECONFIG')]) {
                    sh '''
                    POD_NAME=$(kubectl get pods -n dev -l app=k8s-platform -o jsonpath='{.items[0].metadata.name}')
                    kubectl exec -it $POD_NAME -n dev -- curl -f http://localhost:8080/health
                    '''
                }
            }
        }

        // ===== Staging Deployment =====
        stage('Deploy to Staging') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-staging', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl create namespace staging --dry-run=client -o yaml | kubectl apply -f -
                    kubectl apply -k overlays/staging/ -n staging
                    kubectl rollout status deployment/k8s-platform -n staging --timeout=120s
                    '''
                }
            }
        }

        stage('Staging Health Check') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-staging', variable: 'KUBECONFIG')]) {
                    sh '''
                    POD_NAME=$(kubectl get pods -n staging -l app=k8s-platform -o jsonpath='{.items[0].metadata.name}')
                    kubectl exec -it $POD_NAME -n staging -- curl -f http://localhost:8080/health
                    '''
                }
            }
        }

        // ===== Production Approval =====
        stage('Production Approval') {
            steps {
                input message: 'Approve deployment to PRODUCTION?',
                      ok: 'Deploy',
                      submitter: 'devops-lead,release-manager'
            }
        }

        // ===== Production Deployment =====
        stage('Deploy to Production') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl create namespace prod --dry-run=client -o yaml | kubectl apply -f -
                    kubectl apply -k overlays/prod/ -n prod
                    kubectl rollout status deployment/k8s-platform -n prod --timeout=120s
                    '''
                }
            }
        }

        stage('Production Health Check') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
                    sh '''
                    POD_NAME=$(kubectl get pods -n prod -l app=k8s-platform -o jsonpath='{.items[0].metadata.name}')
                    kubectl exec -it $POD_NAME -n prod -- curl -f http://localhost:8080/health
                    '''
                }
            }
        }

    } // end stages
       // stage('Deploy to Kubernetes') {
         //   steps {
           //     withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
             //       sh '''
               //     kubectl get nodes
                 //   kubectl apply -k overlays/prod/
                   // '''
                //}
            //}
        //}
    
   post {
        success {
            echo 'Deployment completed and verified successfully!'
        }
        failure {
            echo 'Deployment verification failed!'
        }
    }
}
