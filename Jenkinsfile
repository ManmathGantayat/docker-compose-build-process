pipeline {
    agent any

    environment {
        AWS_REGION   = 'us-east-1'
        ECR_REGISTRY = '426192960096.dkr.ecr.us-east-1.amazonaws.com'
        FRONTEND_IMG = 'frontend-app'
        BACKEND_IMG  = 'backend-app'
        MYSQL_IMG    = 'mysql-db'
        DEPLOY_PATH  = '/home/ec2-user/app/docker-compose-build-process'
        EC2_HOST     = '13.218.195.219'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                      aws ecr get-login-password --region $AWS_REGION \
                      | docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Build & Push Backend') {
            steps {
                sh '''
                  docker build -t $ECR_REGISTRY/$BACKEND_IMG:latest backend
                  docker push $ECR_REGISTRY/$BACKEND_IMG:latest
                '''
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh '''
                  docker build -t $ECR_REGISTRY/$FRONTEND_IMG:latest client
                  docker push $ECR_REGISTRY/$FRONTEND_IMG:latest
                '''
            }
        }

        stage('Build & Push MySQL') {
            steps {
                sh '''
                  docker build -t $ECR_REGISTRY/$MYSQL_IMG:latest mysql
                  docker push $ECR_REGISTRY/$MYSQL_IMG:latest
                '''
            }
        }

        stage('Deploy on EC2 (Docker Compose)') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${EC2_HOST} '
                      set -e
                      echo "Connected to EC2"

                      aws sts get-caller-identity

                      aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${ECR_REGISTRY}

                      cd ${DEPLOY_PATH}

                      docker compose pull
                      docker compose down
                      docker compose up -d

                      echo "Deployment completed successfully"
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Jenkins deployment successful"
        }
        failure {
            echo "❌ Jenkins deployment failed"
        }
    }
}
