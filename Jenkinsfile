pipeline {

    agent any

    environment {

        AWS_REGION = 'ap-south-1'

        ECR_REGISTRY = '168286911560.dkr.ecr.ap-south-1.amazonaws.com'

        ECR_REPOSITORY = 'inthu'

        ECR_IMAGE = '168286911560.dkr.ecr.ap-south-1.amazonaws.com/inthu'

        EKS_CLUSTER = 'tcs'

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code already checked out by Jenkins'
            }
        }

        stage('Application Test') {
            steps {
                echo 'Testing Node.js application'

                sh '''
                    npm install
                    node --check app.js
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image'

                sh '''
                    docker build \
                    -t ${ECR_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Test') {
            steps {
                echo 'Testing Docker container'

                sh '''
                    docker rm -f inthu-test 2>/dev/null || true

                    docker run -d \
                    --name inthu-test \
                    -p 8081:8080 \
                    ${ECR_IMAGE}:${IMAGE_TAG}

                    sleep 5

                    curl -f http://localhost:8081/

                    docker stop inthu-test
                    docker rm inthu-test
                '''
            }
        }

        stage('ECR Login') {
            steps {
                echo 'Logging into Amazon ECR'

                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo 'Pushing Docker image to ECR'

                sh '''
                    docker push ${ECR_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Configure EKS') {
            steps {
                echo 'Configuring EKS'

                sh '''
                    aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name ${EKS_CLUSTER}

                    kubectl get nodes
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo 'Deploying application to EKS'

                sh '''
                    sed -i \
                    "s|IMAGE_PLACEHOLDER|${ECR_IMAGE}:${IMAGE_TAG}|g" \
                    k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml

                    kubectl apply -f k8s/service.yaml

                    kubectl rollout status \
                    deployment/inthu \
                    --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying EKS deployment'

                sh '''
                    kubectl get deployment inthu

                    kubectl get pods \
                    -l app=inthu

                    kubectl get service devops-project
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'CI/CD PIPELINE SUCCESS'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'CI/CD PIPELINE FAILED'
            echo '========================================'
        }
    }
}
