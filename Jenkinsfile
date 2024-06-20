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
                    def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', REGISTRY_CREDS) {
                        // Retrieve the built docker image
                        def dockerImage = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Delete Docker Images') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage('updateing kubernetes deployment file'){
            steps{
                script{

                    sh """
                    cat deployment.yml
                    sed -i 's/${APP_NAME}.*/'
                    """
                }
            }
        }
    }
}
