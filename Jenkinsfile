pipeline {
    agent any

    environment {
        REPO = 'jenkins-ci-cd-automation'
        AWS_REGION = 'ap-south-1'
        ECR_URL = "354918399295.dkr.ecr.ap-south-1.amazonaws.com/gaurav-ci-cd"
        EC2_USER = "ubuntu"
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
                    echo "Creating Python venv..."
                    python3 -m venv venv
                    . venv/bin/activate

                    echo "Upgrading pip..."
                    pip install --upgrade pip

                    echo "Installing dependencies..."
                    pip install -r requirements.txt

                    echo "Running tests..."
                    pytest -q || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker Image..."
                    docker build -t gaurav-ci-cd-app .
                '''
            }
        }

        stage('Docker Login to ECR') {
            steps {
                sh '''
                    echo "Logging in to AWS ECR..."
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_URL
                '''
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                    echo "Tagging Image..."
                    docker tag gaurav-ci-cd-app:latest $ECR_URL:latest

                    echo "Pushing Image to ECR..."
                    docker push $ECR_URL:latest
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    echo "Deploying on EC2..."

                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        echo "Pulling latest Docker image..."
                        docker pull $ECR_URL:latest

                        echo "Stopping old container if running..."
                        docker stop ci-cd-app || true

                        echo "Removing old container..."
                        docker rm ci-cd-app || true

                        echo "Starting new container..."
                        docker run -d -p 80:5000 --name ci-cd-app $ECR_URL:latest
                    '
                '''
            }
        }
    }
}

