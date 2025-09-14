pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'docker-creds' // Jenkins credential ID
        DOCKER_IMAGE = "sasank1219/bms"
        SONARQUBE_ENV = 'SonarQube'
        APP_PORT = "3000"
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
                    sh 'npm install --prefer-offline'
                }
            }
        }

        stage('Docker Build & Push to DockerHub') {
            steps {
                dir('bookmyshow-app') {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                     usernameVariable: 'DOCKERHUB_USER',
                                                     passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh '''
                            # Secure Docker login
                            echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USER --password-stdin
                            
                            # Build Docker image
                            docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                            
                            # Push images
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                            docker push ${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                        # Stop existing container if running
                        docker rm -f bms-app || true
                        
                        # Run new container
                        docker run -d --name bms-app -p ${APP_PORT}:${APP_PORT} ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: """
                    Jenkins pipeline succeeded!

                    Job: ${env.JOB_NAME}
                    Build: #${env.BUILD_NUMBER}
                    Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    Application deployed on port ${APP_PORT}.
                """,
                to: "nvssasank1219@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: """
                    Jenkins pipeline failed.

                    Job: ${env.JOB_NAME}
                    Build: #${env.BUILD_NUMBER}

                    Please check Jenkins console logs for details: ${BUILD_URL}
                """,
                to: "nvssasank1219@gmail.com"
            )
        }
    }
}
