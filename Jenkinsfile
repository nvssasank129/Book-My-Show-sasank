pipeline {
    agent any

    environment {
        // DockerHub credentials ID stored in Jenkins
        DOCKERHUB_CREDENTIALS = 'docker-creds'
        // DockerHub repository (replace with your username)
        DOCKER_IMAGE = "sasank1219/bms"

        // SonarQube server (configured in Jenkins > Manage Jenkins > Configure System)
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code from GitHub') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis (Quality Gate)') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=BookMyShow \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Install Dependencies (NPM)') {
            steps {
                dir('bookmyshow-app') {
                    // Use npm install instead of npm ci to avoid lockfile mismatch issues
                    sh 'npm install --prefer-offline'
                }
            }
        }

    
        stage('Docker Build & Push to DockerHub') {
            steps {
                dir('bookmyshow-app') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                                    usernameVariable: 'DOCKERHUB_USERNAME', 
                                                    passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                            docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                            echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }



        stage('Deploy to Docker Container') {
            steps {
                sh '''
                docker rm -f bms-app || true
                docker run -d --name bms-app -p 3000:3000 ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: """
                The Jenkins pipeline succeeded!

                - Job: ${env.JOB_NAME}
                - Build: #${env.BUILD_NUMBER}
                - Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}

                Application deployed on port 3000.
                """,
                to: "nvssasank1219@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: """
                The Jenkins pipeline failed.

                - Job: ${env.JOB_NAME}
                - Build: #${env.BUILD_NUMBER}

                Please check Jenkins console logs for details: ${BUILD_URL}
                """,
                to: "nvssasank1219@gmail.com"
            )
        }
    }
}
