pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'lubern5'
        APP_NAME = 'gitops-argo-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        GIT_CREDENTIALS = 'github'  // Jenkins credential ID for GitHub credentials
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
                    // Checkout the repository from GitHub
                    git credentialsId: GIT_CREDENTIALS,
                        url: 'https://github.com/Lubern5/gitops_argocd_project.git',
                        branch: 'main'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image with the specified tag
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', '') {
                        // Push the Docker image to DockerHub
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
                    // Delete local Docker images to clean up
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Update Deployment File') {
            steps {
                script {
                    // Update deployment.yml with the new Docker image tag
                    sh """
                    sed -i "s|${APP_NAME}:.*|${APP_NAME}:${IMAGE_TAG}|g" deployment.yml
                    cat deployment.yml  // Optional: Print updated deployment.yml for verification
                    """
                }
            }
        }
        
        stage('Commit and Push Deployment File') {
            steps {
                script {
                    try {
                        // Configure Git user details
                        sh """
                        git config --global user.name "lubern5"
                        git config --global user.email "lubern5@yahoo.com"
                        """
                        
                        // Add and commit deployment.yml changes
                        sh "git add deployment.yml"
                        sh "git commit -m 'Update deployment file to ${IMAGE_TAG}'"
                        
                        // Push changes to GitHub repository
                        withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                            sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Lubern5/gitops_argocd_project.git main"
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
