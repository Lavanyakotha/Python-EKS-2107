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
                git url: 'https://github.com/Lavanyakotha/Python-EKS-2107.git',
                     credentialsId: 'github-credentials',
                     branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t flask-app ./app'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
        
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
        
                        docker tag flask-app:latest $ECR_REPO:$IMAGE_TAG
                        docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }


        stage('Deploy to EKS') {
            steps {
                 withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    script {
                        sh '''
                            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                            aws configure set default.region $AWS_REGION
        
                            # Update kubeconfig for EKS
                            aws eks update-kubeconfig --region $AWS_REGION --name <your-cluster-name>
        
                            # Deploy application
                            kubectl apply -f k8s/flask-app.yaml
                        '''
                    }
                }
            }
        }
    }
}
