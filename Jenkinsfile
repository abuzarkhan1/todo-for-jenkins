pipeline {
    agent any
    tools{
        nodejs 'node'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/abuzarkhan1/todo-for-jenkins.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('jenkins-react-app', '.')
                }
            }
        }
        stage('Run Docker Image') {
            steps {
                script {
                    docker.image('jenkins-react-app').run('-p 3000:3000')
                }
            }
        }
    }
}
