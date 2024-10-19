pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar'
        IMAGE_VERSION = "${env.BUILD_NUMBER}"  // Using build number as image version
        DOCKER_IMAGE = "abuzarkhan1/todo:${IMAGE_VERSION}"
        GITHUB_CREDENTIALS = credentials('GITHUB') // Replace with your GitHub credentials ID
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/abuzarkhan1/todo-for-jenkins.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Todo \
                        -Dsonar.projectName=Todo'''
                }
            }
        }
        stage('Trivy File System Scan') {
            steps {
                script {
                    try {
                        sh "trivy fs . > trivyfs.txt"
                    } catch (Exception e) {
                        echo "Trivy FS scan failed: ${e.message}"
                        error "Stopping pipeline due to Trivy FS scan failure."
                    }
                }
            }
        }
        stage('Docker Build Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image ${DOCKER_IMAGE} > trivyimage.txt"
            }
        }
        stage('Check Docker Version') {
            steps {
                sh 'docker --version'
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        stage('Update Kubernetes Deployment YAML') {
            steps {
                script {
                    // Update the image version in Deploy.yaml
                    dir('k8s') {
                        sh """
                        sed -i 's|image: abuzarkhan1/todo:.*|image: ${DOCKER_IMAGE}|' Deploy.yaml
                        """
                    }
                }
            }
        }
        stage('Commit and Push Changes') {
            steps {
                script {
                    dir('k8s') {
                        sh '''
                        echo "Checking repository status:"
                        git status
                        
                        echo "Configuring Git user:"
                        git config user.name "Abuzar"
                        git config user.email "abuzarkhan1242@gmail.com"
                        
                        echo "Adding changes to git:"
                        git add Deploy.yaml
                        
                        echo "Committing changes:"
                        git commit -m "Update Deploy.yaml with new image version ${IMAGE_VERSION}" || true
                        
                        echo "Pushing changes to GitHub:"
                        git push https://${GITHUB_CREDENTIALS}@github.com/abuzarkhan1/todo-for-jenkins.git HEAD:master
                        '''
                    }
                }
            }
        }
    }
}
