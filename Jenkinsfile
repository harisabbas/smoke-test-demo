pipeline {

    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "570220875158"
        ECR_REPOSITORY = "smoke-demo"

        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:latest"
    }

    stages {

        stage('Clone Repository') {
            steps {

                git branch: 'main',
                    url: 'https://github.com/harisabbas/smoke-test-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {

                sh '''
                    docker build -t smoke-demo .
                '''
            }
        }

        stage('Run Container') {
            steps {

                sh '''
                    docker rm -f smoke-test || true

                    docker run -d \
                        --name smoke-test \
                        -p 8086:80 \
                        smoke-demo
                '''
            }
        }

        stage('Smoke Test') {
            steps {

                sh '''
                    sleep 5
                    curl http://localhost:8086
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {

                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login \
                        --username AWS \
                        --password-stdin \
                        $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {

                sh '''
                    docker tag smoke-demo:latest $IMAGE_URI
                '''
            }
        }

        stage('Push Docker Image') {
            steps {

                sh '''
                    docker push $IMAGE_URI
                '''
            }
        }
    }

    post {

        success {
            echo 'Docker Image Built, Tested and Pushed to ECR Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }

        always {
            echo 'Pipeline Execution Completed'
        }
    }
}


