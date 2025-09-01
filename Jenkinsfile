pipeline {
    agent any

    environment {
        DOCKER_REPO = "anand20003/project2"
        AWS_REGION  = "us-east-1"
        IMAGE_TAG   = "${env.BUILD_NUMBER}"   
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Anand-kumar-git/Project2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "Building Docker image..."
                    docker build -t ${DOCKER_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKER_REPO}:${IMAGE_TAG} ${DOCKER_REPO}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:latest
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-creds']]) {
                    sh """
                        echo "Updating kubeconfig..."
                        aws eks update-kubeconfig --region ${AWS_REGION} --name my-cluster

                        echo "Deploying to Kubernetes..."
                        kubectl set image deployment/myapp-deployment myapp=${DOCKER_REPO}:${IMAGE_TAG} || true

                        # Apply Kubernetes manifests from k8s folder
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed. Check logs in Jenkins!"
        }
    }
}
