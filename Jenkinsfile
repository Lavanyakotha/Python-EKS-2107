pipeline {
    agent any

environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '076194731919.dkr.ecr.us-east-1.amazonaws.com/flask-app'
        IMAGE_TAG = "latest"
    }
    

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/lavanyakotha/python-eks-2017.git',
                     credentialsId: 'github-credentials',
                     branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t flask-app .'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    docker tag flask-app:latest $ECR_REPO:$IMAGE_TAG
                    docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
