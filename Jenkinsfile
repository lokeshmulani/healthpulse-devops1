pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '133197206805.dkr.ecr.us-east-1.amazonaws.com/healthpulse-frontend'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE = "${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker image: ${IMAGE}"
                    docker build -t ${IMAGE} app/
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_REPO}
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    echo "Pushing ${IMAGE}"
                    docker push ${IMAGE}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    kubectl -n healthpulse set image deployment/frontend \
                    frontend=${IMAGE}
                '''
            }
        }

        stage('Rollout Status') {
            steps {
                sh '''
                    kubectl -n healthpulse rollout status deployment/frontend --timeout=180s
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    kubectl -n healthpulse get pods
                    kubectl -n healthpulse get service frontend-service
                '''
            }
        }
    }

    post {
        success {
            echo 'HealthPulse deployment completed successfully!'
        }

        failure {
            echo 'HealthPulse deployment failed. Check the Jenkins console output.'
        }
    }
}
