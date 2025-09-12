pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_USERNAME = 'jawahar11'
        IMAGE_NAME = 'trend-static-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        EKS_CLUSTER_NAME = 'trendify-eks-cluster'
        AWS_REGION = 'ap-south-1'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    def dockerImage = docker.build("${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}")
                    docker.build("${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo 'Pushing Docker image to DockerHub...'
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        def dockerImage = docker.image("${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}")
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS Creds']]) {
                    script {
                        echo 'Updating Kubernetes deployment...'
                        sh """
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                            kubectl set image deployment/trendify-deployment trendify-container=${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} -n trendify
                            kubectl rollout status deployment/trendify-deployment -n trendify
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS Creds']]) {
                    script {
                        echo 'Verifying deployment...'
                        sh """
                            kubectl get pods -n trendify
                            kubectl get services -n trendify
                            echo 'Deployment verification completed'
                        """
                    }
                }
            }
        }
    } 

    post {
        always {
            echo 'Pipeline execution completed'
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
    }
} 
