pipeline {

    agent any

    environment {

        AWS_REGION = 'ap-south-1'

        ECR_REGISTRY = '467765330161.dkr.ecr.ap-south-1.amazonaws.com'

        ECR_REPOSITORY = 'devops-project'

        ECR_IMAGE = '467765330161.dkr.ecr.ap-south-1.amazonaws.com/devops-project'

        EKS_CLUSTER = 'tcs'

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {

                echo 'Checking out source code'

                git(
                    branch: 'master',
                    url: 'https://github.com/NAVITECHDEVOPS/sample-node-app.git'
                )
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
                    docker run -d \
                    --name devops-project-test \
                    -p 8080:8080 \
                    ${ECR_IMAGE}:${IMAGE_TAG}

                    sleep 5

                    curl -f http://localhost:8080/

                    docker stop devops-project-test
                    docker rm devops-project-test
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
                    deployment/devops-project \
                    --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {

                echo 'Verifying EKS deployment'

                sh '''
                    kubectl get deployment devops-project

                    kubectl get pods \
                    -l app=devops-project

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
