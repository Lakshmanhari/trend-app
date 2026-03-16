pipeline {
    agent any

    stages {

        stage('Clone Repo') {
            steps {
                git 'https://github.com/Lakshmanhari/trend-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t lakshmanhari/trend-app:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh 'docker push lakshmanhari/trend-app:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

    }
}