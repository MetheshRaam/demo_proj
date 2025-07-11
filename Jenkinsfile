pipeline {
    agent any

    environment {
        IMAGE_NAME = "metheshregistry.azurecr.io/demo-app1:latest"
        ACR_LOGIN_SERVER = "metheshregistry.azurecr.io"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MetheshRaam/demo_proj.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME% .'
            }
        }

        stage('Push Image to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    bat '''
                    docker login %ACR_LOGIN_SERVER% -u %USERNAME% -p %PASSWORD%
                    docker push %IMAGE_NAME%
                    '''
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG_FILE%
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }
}
