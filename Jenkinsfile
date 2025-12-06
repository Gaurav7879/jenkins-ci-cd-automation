pipeline {
    agent any

    environment {
        REPO = 'jenkins-ci-cd-automation'
        AWS_REGION = 'ap-south-1'
        ECR_URL = "311122233344.dkr.ecr.ap-south-1.amazonaws.com/gaurav-ci-cd"
        EC2_USER = "ec2-user"
        EC2_IP = "YOUR_EC2_PUBLIC_IP"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: "https://github.com/Gaurav7879/${REPO}.git"
            }
        }

        stage('Install Dependencies & Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest -q'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t gaura-ci-cd-app .'
            }
        }

        stage('Docker Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $ECR_URL
                '''
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                docker tag gaura-ci-cd-app:latest $ECR_URL:latest
                docker push $ECR_URL:latest
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                    docker pull $ECR_URL:latest &&
                    docker stop ci-cd-app || true &&
                    docker rm ci-cd-app || true &&
                    docker run -d -p 80:5000 --name ci-cd-app $ECR_URL:latest
                '
                '''
            }
        }
    }
}

