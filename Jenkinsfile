pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "orderservice"
        ECS_CLUSTER = "ramya-cluster1"
        ECS_SERVICE = "orderservice-task-service-ouhxa3tt"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_ACCOUNT_ID = "792520190237"
        ECR_URI = "792520190237.dkr.ecr.us-east-1.amazonaws.com/orderservice"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:latest
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker push ${ECR_URI}:${IMAGE_TAG}
                docker push ${ECR_URI}:latest
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh """
                aws ecs update-service \
                    --cluster ${ECS_CLUSTER} \
                    --service ${ECS_SERVICE} \
                    --force-new-deployment \
                    --region ${AWS_REGION}
                """
            }
        }
    }

    post {
        always {
            sh "docker image prune -f"
        }
    }
}
