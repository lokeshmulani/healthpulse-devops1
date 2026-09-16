pipeline {
agent any

```
environment {
    AWS_REGION = 'us-east-1'
    ECR_REPO = '133197206805.dkr.ecr.us-east-1.amazonaws.com/healthpulse-frontend'
    IMAGE_TAG = "${BUILD_NUMBER}"
    IMAGE = "${ECR_REPO}:${IMAGE_TAG}"
}

options {
    skipDefaultCheckout(true)
}

stages {

    stage('Checkout') {
        steps {
            git branch: 'master',
                url: 'https://github.com/lokeshmulani/healthpulse-devops1.git'
        }
    }

    stage('Docker Build') {
        steps {
            sh '''
                docker build -t ${IMAGE} app/
            '''
        }
    }

    stage('ECR Login') {
        steps {
            sh '''
                aws ecr get-login-password --region ${AWS_REGION} \
                | docker login --username AWS --password-stdin 133197206805.dkr.ecr.us-east-1.amazonaws.com
            '''
        }
    }

    stage('Push Image') {
        steps {
            sh '''
                docker push ${IMAGE}
            '''
        }
    }

    stage('Deploy to EKS') {
        steps {
            sh '''
                kubectl -n healthpulse set image deployment/frontend frontend=${IMAGE}
            '''
        }
    }

    stage('Verify Rollout') {
        steps {
            sh '''
                kubectl -n healthpulse rollout status deployment/frontend --timeout=180s
            '''
        }
    }

    stage('Health Check') {
        steps {
            sh '''
                kubectl -n healthpulse get pods -o wide
                kubectl -n healthpulse get svc
            '''
        }
    }
}

post {
    success {
        echo 'Deployment successful'
    }

    failure {
        echo 'Deployment failed - rolling back'

        sh '''
            kubectl -n healthpulse rollout undo deployment/frontend || true
        '''
    }
}
```

}
