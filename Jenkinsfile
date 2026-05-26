pipeline {
    agent any

    environment {
        IMAGE_NAME = 'aakashchauhan27/devops-jenkins-demo'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT[0..6]}"
        LATEST_TAG = "${IMAGE_NAME}:latest"
        DOCKERHUB_CRED = 'dockerhub-credentials'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Building commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Build & Test') {
            steps {

                dir('app') {

                    bat 'mvn clean test'

                    bat 'mvn package -DskipTests -q'
                }
            }

            post {
                always {

                    junit 'app/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {

                bat "docker build -t ${IMAGE_TAG} -t ${LATEST_TAG} ."

                bat "docker images | findstr devops-jenkins-demo"
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CRED}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {

                    bat 'echo %DOCKER_TOKEN% | docker login -u %DOCKER_USER% --password-stdin'

                    bat "docker push ${IMAGE_TAG}"

                    bat "docker push ${LATEST_TAG}"

                    bat 'docker logout'
                }
            }
        }

        stage('Verify Deployment') {
            steps {

                bat 'docker stop test-app || exit 0'

                bat 'docker rm test-app || exit 0'

                bat "docker run -d --name test-app -p 8081:8080 ${LATEST_TAG}"

                bat 'timeout /t 15'

                bat 'curl http://localhost:8081/health'
            }

            post {
                always {

                    bat 'docker stop test-app'

                    bat 'docker rm test-app'
                }
            }
        }
    }

    post {

        success {

            echo 'Pipeline SUCCESS — Docker image pushed successfully!'
        }

        failure {

            echo 'Pipeline FAILED — check logs for details.'
        }
    }
}
