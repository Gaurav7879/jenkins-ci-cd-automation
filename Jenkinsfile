pipeline {
    agent any

    environment {
        REPO = 'jenkins-ci-cd-automation'
        AWS_REGION = 'ap-south-1'
        ECR_URL = "354918399295.dkr.ecr.ap-south-1.amazonaws.com/gaurav-ci-cd"
        EC2_USER = "root"
        EC2_IP = "3.111.35.94"
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
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pytest -q || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t gaurav-ci-cd-app .
                '''
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
                    docker tag gaurav-ci-cd-app:latest $ECR_URL:latest
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

