pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'anand20003'
        DOCKER_IMAGE = "${DOCKERHUB_USER}/project2"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Anand-kumar-git/Project2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .'
            }
        }

        stage('Push To DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Update kubeconfig for EKS cluster
                aws eks update-kubeconfig --region us-east-1 --name myapp-cluster

                # Apply manifests
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml

                # Update deployment image
                kubectl set image deployment/myapp-deployment myapp=$DOCKER_IMAGE:${BUILD_NUMBER} --record

                # Verify rollout
                kubectl rollout status deployment/myapp-deployment
                '''
            }
        }
    }
}
