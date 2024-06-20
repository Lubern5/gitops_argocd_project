pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'lubern5'
        APP_NAME = 'gitops-argo-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub' // Jenkins credential ID for DockerHub credentials
        GIT_CREDENTIALS = 'github' // Jenkins credential ID for Git credentials
        GIT_REPO_URL = 'https://github.com/Lubern5/gitops_argocd_project.git'
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
                    def docker_image
                    try {
                        docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    } catch (Exception e) {
                        echo "Failed to build Docker image: ${e.message}"
                        error "Build failed"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    try {
                        docker.withRegistry('', REGISTRY_CREDS) {
                            // Retrieve the built docker image
                            def dockerImage = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")
                            dockerImage.push()
                            dockerImage.push('latest')
                        }
                    } catch (Exception e) {
                        echo "Failed to push Docker image: ${e.message}"
                        error "Push failed"
                    }
                }
            }
        }
        
        stage('Delete Docker Images') {
            steps {
                script {
                    try {
                        sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker rmi ${IMAGE_NAME}:latest"
                    } catch (Exception e) {
                        echo "Failed to delete Docker images: ${e.message}"
                    }
                }
            }
        }
        
        stage('Updating Kubernetes Deployment File') {
            steps {
                script {
                    try {
                        sh """
                        sed -i "s|${APP_NAME}.*|${APP_NAME}:${IMAGE_TAG}|g" deployment.yml
                        cat deployment.yml
                        cat service.yml
                        """
                    } catch (Exception e) {
                        echo "Failed to update Kubernetes deployment file: ${e.message}"
                        error "Update failed"
                    }
                }
            }
        }
        
        stage('Push the changed deployment file to Git') {
            steps {
                script {
                    try {
                        git config --global user.name "lubern5"
                        git config --global user.email "lubern5@yahoo.com"
                        git add deployment.yml
                        git commit -m "updated the deployment file"
                        
                        withCredentials([gitUsernamePassword(credentialsId: GIT_CREDENTIALS, gitToolName: 'Default')]) {
                            sh "git push ${GIT_REPO_URL} main"
                        }
                    } catch (Exception e) {
                        echo "Failed to push changes to Git: ${e.message}"
                        error "Git push failed"
                    }
                }
            }
        }
    }
}
