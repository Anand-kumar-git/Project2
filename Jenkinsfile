pipeline {
    agent any

    environment {
        DOCKER_REPO = "anand20003/project2"
        AWS_REGION = "us-east-1"
        IMAGE_TAG = "${env.BUILD_NUMBER}"   // unique tag for each build
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
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
                sh """
                echo "Updating kubeconfig..."
                aws eks update-kubeconfig --region ${AWS_REGION} --name my-cluster

                echo "Updating Kubernetes Deployment..."
                kubectl set image deployment/myapp-deployment myapp=${DOCKER_REPO}:${IMAGE_TAG} --record || true

                echo "Applying manifests..."
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed. Check logs in Jenkins!"
        }
    }
}
