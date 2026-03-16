pipeline {
    agent any

    stages {

        stage('Clone Repo') {
            steps {
               git branch: 'main', url: 'https://github.com/Lakshmanhari/trend-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t lakshmanhari/trend-app:latest .'
            }
        }

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

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

    }
}