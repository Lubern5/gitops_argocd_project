pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "lubern5"
        APP_NAME = "gitops-argo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }
    
    stages {
        stage('Cleanup workspace') {
            steps {
                deleteDir()
            }
        }
        
        stage('Checkout SCM') {
            steps {
                script {
                    git credentialsId: 'github',
                        url: 'https://github.com/Lubern5/gitops_argocd_project',
                        branch: 'main'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image with the specified tag
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    // Store dockerImage as a global variable
                    env.DOCKER_IMAGE = dockerImage
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to the registry
                    docker.withRegistry('', REGISTRY_CREDS) {
                        // Use the global variable for dockerImage
                        env.DOCKER_IMAGE.push("${IMAGE_TAG}")
                        env.DOCKER_IMAGE.push('latest')
                    }
                }
            }
        }
        
        stage('Delete Docker Images') {
            steps {
                script {
                    // Remove the local Docker images (optional step)
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
    }
}
