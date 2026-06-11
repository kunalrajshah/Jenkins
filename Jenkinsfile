pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java21'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "shahkunal0709"
        DOCKER_PASS = 'dockerhub'          // Jenkins DockerHub credentials ID
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'Github',
                    url: 'https://github.com/kunalrajshah/end-to-end-cicd-pipeline.git'
            }
        }

        stage("Build Application") {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("Test Application") {
            steps {
                sh 'mvn test'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'Jenkins-sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false,  credentialsId: 'Jenkins-sonarqube-token'
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {

                        def dockerImage = docker.build("${IMAGE_NAME}")

                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh """
                docker run --rm \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  aquasec/trivy image \
                  ${IMAGE_NAME}:latest \
                  --no-progress \
                  --scanners vuln \
                  --exit-code 0 \
                  --severity HIGH,CRITICAL \
                  --format table
                """
            }
        }

        stage("Deploy Application") {
            steps {
                sh """
                echo "Stopping existing container if running..."
                docker stop register-app || true

                echo "Removing existing container if present..."
                docker rm register-app || true

                echo "Removing old image..."
                docker rmi ${IMAGE_NAME}:latest || true

                echo "Pulling latest image from Docker Hub..."
                docker pull ${IMAGE_NAME}:latest

                echo "Running new container..."
                docker run -d \
                    --name register-app \
                    -p 8080:80 \
                    ${IMAGE_NAME}:latest

                echo "Deployment completed successfully."
                """
            }
        }
    }
}
