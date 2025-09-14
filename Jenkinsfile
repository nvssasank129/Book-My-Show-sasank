pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "sasank1219/bms"
        APP_PORT = 3000
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'feature_devops_setup', url: 'https://github.com/Sejalkarwa/Book-My-Show.git'
                sh 'ls -la'  // Verify files after checkout
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """ 
                    $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=BMS \
                        -Dsonar.projectKey=BMS
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh """
                    if [ -f package.json ]; then
                        rm -rf node_modules package-lock.json
                        npm install --prefer-offline
                    else
                        echo "Error: package.json not found!"
                        exit 1
                    fi
                    """
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "Logging into DockerHub..."
                        docker login -u $DOCKER_USER -p $DOCKER_PASS

                        echo "Building Docker image..."
                        docker build --no-cache -t ${DOCKER_IMAGE}:latest -f bookmyshow-app/Dockerfile bookmyshow-app

                        echo "Pushing Docker image..."
                        docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh """
                echo "Stopping old container if exists..."
                docker rm -f bms || true

                echo "Running new container..."
                docker run -d --restart=always --name bms -p ${APP_PORT}:${APP_PORT} ${DOCKER_IMAGE}:latest

                echo "Checking running containers..."
                docker ps -a

                echo "Fetching logs..."
                sleep 5
                docker logs bms
                """
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                subject: "Build ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Project: ${env.JOB_NAME}<br/>
                Build Number: ${env.BUILD_NUMBER}<br/>
                URL: ${env.BUILD_URL}<br/>
                """,
                to: 'nvssasank1219@gmail.com'
        }
    }
}
