pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "samimmondal/sample"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-cred') {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // This assumes kubectl is configured on the Jenkins agent
                    sh """
                    kubectl set image deployment/sm-deploy \
                      sm-deploy=${DOCKER_IMAGE}:${IMAGE_TAG} \
                      --namespace=default
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment succeeded!"
        }
        failure {
            echo "❌ Something went wrong."
        }
    }
}
