pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "leocrita/java-ecom-app:latest"
        K8S_DIR = "k8s"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/preciouschidera/java-ecom-eks-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig --region eu-west-2 --name ecommerce-eks
                    kubectl apply -f terraform/k8s/deployment.yaml
                    kubectl apply -f terraform/k8s/service.yaml
                    kubectl apply -f terraform/k8s/ingress.yaml
                '''
            }
        }
    }
}

