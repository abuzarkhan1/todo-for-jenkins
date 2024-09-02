pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/abuzarkhan1/todo-for-jenkins.git'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Install') {
            steps {
                sh 'npm install additional-package'
            }
        }
    }
}
