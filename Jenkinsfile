pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "anand20003/project2"
        KUBECONFIG_CREDENTIALS = credentials('kubeconfig-cred')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Anand-kumar-git/Project2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
                sh 'docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest'
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
                sh 'docker push $DOCKER_IMAGE:latest'
            }
        }

        stage('Test Kubeconfig') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG'),
                    usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    export AWS_REGION=us-east-1
                    kubectl get nodes --kubeconfig=$KUBECONFIG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG'),
                    usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    export AWS_REGION=us-east-1
                    kubectl set image deployment/myapp-deployment myapp=$DOCKER_IMAGE:$BUILD_NUMBER --kubeconfig=$KUBECONFIG
                    kubectl rollout status deployment/myapp-deployment --kubeconfig=$KUBECONFIG
                    '''
                }
            }
        }
    }
}
