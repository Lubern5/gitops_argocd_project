pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'lubern5'
        APP_NAME = 'gitops-argo-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub' // Jenkins credential ID for DockerHub credentials
        GIT_CREDENTIALS = 'github' // Jenkins credential ID for Git credentials
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
                    git credentialsId: 'github',
                        url: 'https://github.com/Lubern5/gitops_argocd_project',
                        branch: 'main'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', REGISTRY_CREDS) {
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
        
        stage('Updating Kubernetes Deployment File') {
            steps {
                script {
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
                        sh """
                        git config --global user.name "lubern5"
                        git config --global user.email "lubern5@yahoo.com"
                        git add deployment.yml
                        git commit -m "Update deployment file to ${IMAGE_TAG}"
                        """
                        
                        withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
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
