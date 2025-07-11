pipeline {
    agent any

    environment {
        IMAGE_NAME = "metheshregistry.azurecr.io/demo-app1:latest"
        ACR_LOGIN_SERVER = "metheshregistry.azurecr.io"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/MetheshRaam/demo_proj.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push Image to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'docker login $ACR_LOGIN_SERVER -u $USERNAME -p $PASSWORD'
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh 'export KUBECONFIG=$KUBECONFIG_FILE'
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
