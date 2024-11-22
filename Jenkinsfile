def jenkinsNode = "Jenkins-Agent"

withCredentials([string(credentialsId: 'JENKINS_AWS_ACCESS_KEY_ID', variable: 'my_secret_value')]) {
                // Your pipeline steps that use the secret go here
                // The secret value is available in the variable "my_secret_value"
                AWS_ACCESS_KEY_ID = "${my_secret_value}"
                }
withCredentials([string(credentialsId: 'JENKINS_AWS_SECRET_ACCESS_KEY', variable: 'my_secret_value')]) {
                // Your pipeline steps that use the secret go here
                // The secret value is available in the variable "my_secret_value"
                AWS_SECRET_ACCESS_KEY = "${my_secret_value}"
                }

pipeline {
    agent { label "${jenkinsNode}" }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        AWS_ACCOUNT_ID = "231791477922"
        AWS_REGION = "ap-northeast-1"
        IMAGE_NAME = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${APP_NAME}"
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
                git branch: 'backup', credentialsId: 'github', url: 'https://github.com/fahadriaz1206/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gates") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Authenticate with AWS ECR") {
            steps {
                script {
                    sh """
                    set +x
                    export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                    export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    set -x
                    """
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Build the Docker image
                    docker_image = docker.build "${IMAGE_NAME}:${IMAGE_TAG}"

                    // Tag and Push the image to ECR
                    sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}

                    """
                }
            }
        }
    }
}