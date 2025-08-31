pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'anand20003'
        DOCKER_IMAGE = "${DOCKERHUB_USER}/project2"
        AWS_REGION = 'us-east-1'
        CLUSTER_NAME = 'myapp-cluster'    
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

        stage('Configure kubeconfig') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
                        kubectl get nodes
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml

                kubectl set image deployment/myapp-deployment myapp=$DOCKER_IMAGE:${BUILD_NUMBER} --record
                kubectl rollout status deployment/myapp-deployment
                '''
            }
        }
    }
}
