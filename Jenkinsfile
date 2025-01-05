pipeline {
    agent none
    environment {
        AWS_REGION = 'us-east-1' // Replace with your AWS region
        ECR_REPOSITORY = 'ninja' // Replace with your ECR repository name
        IMAGE_TAG = "${env.BUILD_ID}" // Tag image with Jenkins build ID
        AWS_ACCOUNT_ID = '524771228897' // Replace with your AWS account ID
        EC2_USER = 'ubuntu' // Default EC2 username
        EC2_IP = '10.0.4.8' // Replace with your EC2 private IP
    }
    stages {
        stage('Build Docker Image') {
        agent { label 'app' }
            steps {
                sh '''
                echo "Building Docker image..."
                docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
                '''
            }
        }
        stage('Push to ECR') {
        agent { label 'app' }
            steps {
                sh '''
                echo "Logging in to ECR..."
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                
                echo "Tagging and pushing Docker image to ECR..."
                docker tag $ECR_REPOSITORY:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG
                '''
            }
        }
        stage('Deploy on EC2') {
        agent { label 'app' }
            steps {
                sh '''
                #echo "Connecting to EC2 instance and deploying the container..."
                #ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP << EOF
                docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com <<< $(aws ecr get-login-password --region $AWS_REGION)
                docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG
                docker stop upg-app-1 || true
                docker rm upg-app-1 || true
                docker run -d -p 8081:8081 --name upg-app-1 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG
                EOF
                '''
            }
        }
    }
    post {
        success {
            echo 'Application successfully built, pushed, and deployed!'
        }
        failure {
            echo 'Something went wrong. Check the logs.'
        }
    }
}
