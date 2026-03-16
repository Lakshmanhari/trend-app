pipeline {
    agent any

    stages {

        // ---------------- Clone Repo ----------------
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Lakshmanhari/trend-app.git'
            }
        }

        // ---------------- Build Docker Image ----------------
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t lakshmanhari/trend-app:latest .'
            }
        }

        // ---------------- Push to DockerHub ----------------
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push lakshmanhari/trend-app:latest
                    '''
                }
            }
        }

        // ---------------- Configure AWS & Kubeconfig ----------------
        stage('Configure AWS & Kubeconfig') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-eks-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        aws eks update-kubeconfig --region ap-south-1 --name trend-eks-cluster
                    '''
                }
            }
        }

        // ---------------- Deploy to Kubernetes ----------------
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/trend-app
                '''
            }
        }

    }
}