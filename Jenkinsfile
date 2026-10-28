pipeline {
    agent any

    environment {
        IMAGE = 'anand1306/jenkins-k8s-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE%:%BUILD_NUMBER% .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                    bat 'docker push %IMAGE%:%BUILD_NUMBER%'
                    bat 'docker logout'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl set image deployment/demo-app demo-app=%IMAGE%:%BUILD_NUMBER%'
                bat 'kubectl rollout status deployment/demo-app --timeout=120s'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed to Kubernetes!'
        }
        failure {
            echo 'FAILED: Check the Jenkins Console Output.'
        }
    }
}
