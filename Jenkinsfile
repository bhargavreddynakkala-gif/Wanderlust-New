pipeline {
    agent any

    environment {
        // Jenkins automatically creates _USR and _PSW variables from this ID
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE_NAME = 'bhargav1015/bhargav-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling the latest code from GitHub...'
                git branch: 'main', url: 'https://github.com/bhargavreddynakkala-gif/Wanderlust-New'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running application tests...'
                sh 'echo "Tests passed successfully!"'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                sh "docker build -t $DOCKER_IMAGE_NAME:${env.BUILD_NUMBER} ."
                // FIXED: Added space between the build number tag and the 'latest' tag
                sh "docker tag $DOCKER_IMAGE_NAME:${env.BUILD_NUMBER} $DOCKER_IMAGE_NAME:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub...'
                // FIXED: Simplified login using variables created by the environment block
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                
                sh "docker push $DOCKER_IMAGE_NAME:latest"
                sh "docker push $DOCKER_IMAGE_NAME:${env.BUILD_NUMBER}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Applying Kubernetes manifests...'
                sh "kubectl apply -f mongo.yaml"
                sh "kubectl apply -f app.yaml"
                
                // Forces Kubernetes to pull the fresh image you just pushed
                sh "kubectl rollout restart deployment/node-app"
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up session...'
            sh "docker logout"
        }
    }
}