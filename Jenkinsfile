pipeline {
    agent any
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        def app = docker.build("your-dockerhub-username/bms:${BUILD_NUMBER}")
                        app.push()
                    }
                }
            }
        }
    }
    post {
        success {
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: "The build succeeded!",
                to: "your-email@example.com"
            )
        }
        failure {
            emailext(
                subject: "FAILED: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: "The build failed!",
                to: "your-email@example.com"
            )
        }
    }
}
