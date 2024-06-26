pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'lubern5'
        APP_NAME = 'gitops-argo-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        GIT_CREDENTIALS = 'github' // Jenkins credential ID for GitHub credentials
        GIT_REPO_URL = 'https://github.com/Lubern5/gitops_argocd_project.git' // Git repository URL
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
                    // Checkout the Git repository using specified credentials
                    checkout([$class: 'GitSCM',
                              branches: [[name: 'main']],
                              userRemoteConfigs: [[url: "${GIT_REPO_URL}", credentialsId: "${GIT_CREDENTIALS}"]]])
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to DockerHub
                    docker.withRegistry('', 'dockerhub') {
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
                    // Clean up Docker images
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Updating Kubernetes Deployment File') {
            steps {
                script {
                    // Update deployment file with new Docker image tag
                    sh """
                    sed -i "s|${APP_NAME}.*|${APP_NAME}:${IMAGE_TAG}|g" deployment.yml
                    cat deployment.yml
                    """
                }
            }
        }
        
        stage('Commit and Push Deployment File') {
            steps {
                script {
                    try {
                        // Configure Git user info (if not already configured globally)
                        sh """
                        git config --global user.name "lubern5"
                        git config --global user.email "lubern5@yahoo.com"
                        """

                        // Add, commit, and push deployment file changes
                        sh "git add deployment.yml"
                        sh "git commit -m 'Update deployment file to ${IMAGE_TAG}'"

                        // Push changes to GitHub repository
                        sh "git push ${GIT_REPO_URL} main"
                    } catch (Exception e) {
                        echo "Failed to push changes to Git: ${e.message}"
                        error "Git push failed"
                    }
                }
            }
        }
    }
}
