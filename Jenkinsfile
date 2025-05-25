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
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
                        // Write kubeconfig content to file on agent
                        writeFile file: 'kubeconfig', text: KUBECONFIG_CONTENT
                        // Set KUBECONFIG env variable so kubectl uses this file
                        env.KUBECONFIG = "${pwd()}/kubeconfig"

                        sh """
                            kubectl set image deployment/trial samimmondal=${DOCKER_IMAGE}:${IMAGE_TAG} --namespace=default
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded! Deployed ${DOCKER_IMAGE}:${IMAGE_TAG} to Kubernetes."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
