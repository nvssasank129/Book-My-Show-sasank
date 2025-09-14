pipeline {
    agent any

    environment {
        // DockerHub credentials ID stored in Jenkins
        DOCKERHUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = "sasank1219/bms"

        // SonarQube server configured in Jenkins
        SONARQUBE_ENV = 'SonarQube'
    }

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

        stage('SonarQube Analysis') {
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

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh 'npm install --prefer-offline'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                dir('bookmyshow-app') {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS}",
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
                    )]) {
                        sh '''
                            echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                            docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
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
                        docker rm -f bms-app || true
                        docker run -d --name bms-app -p 3000:3000 ${DOCKER_IMAGE}:latest
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
                The Jenkins pipeline succeeded!

                - Job: ${env.JOB_NAME}
                - Build: #${env.BUILD_NUMBER}
                - Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}

                Application deployed on port 3000.
                """,
                to: "nvssasank129@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                body: """
                The Jenkins pipeline failed.

                - Job: ${env.JOB_NAME}
                - Build: #${env.BUILD_NUMBER}

                Check Jenkins console logs for details: ${BUILD_URL}
                """,
                to: "nvssasank129@gmail.com"
            )
        }
    }
}
