pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar'  
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
                sh "docker build -t todo ."
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image todo > trivyimage.txt"
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
                        sh "docker tag todo abuzarkhan1/todo:latest"
                        sh "docker push abuzarkhan1/todo:latest"
                    }
                }
            }
        }
    }
}
