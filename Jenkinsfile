pipeline {
    agent any

    environment {
        IMAGE_NAME = "aakashchauhan27/devops-jenkins-demo"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKERHUB_CRED = "dockerhub-credentials"
    }

    stages {

        // =========================================
        // Stage 1 : Checkout Source Code
        // =========================================
        stage('Checkout') {
            steps {

                checkout scm

                echo "Building Commit: ${env.GIT_COMMIT}"
            }
        }

        // =========================================
        // Stage 2 : Build and Test
        // =========================================
        stage('Build & Test') {
            steps {

                dir('app') {

                    bat 'mvn clean test'

                    bat 'mvn package -DskipTests'
                }
            }

            post {
                always {

                    junit 'app/target/surefire-reports/*.xml'
                }
            }
        }

        // =========================================
        // Stage 3 : Docker Build
        // =========================================
        stage('Docker Build') {
            steps {

                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% ."

                bat "docker images"
            }
        }

        // =========================================
        // Stage 4 : Docker Push
        // =========================================
        stage('Docker Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'

                    bat "docker push %IMAGE_NAME%:%IMAGE_TAG%"

                    bat 'docker logout'
                }
            }
        }

        // =========================================
        // Stage 5 : Verify Deployment
        // =========================================
        stage('Verify Deployment') {
            steps {

                // Stop old container if exists
                bat 'docker stop test-app >nul 2>&1 || exit 0'

                // Remove old container if exists
                bat 'docker rm test-app >nul 2>&1 || exit 0'

                // Run container
                bat "docker run -d --name test-app -p 9090:8080 %IMAGE_NAME%:%IMAGE_TAG%"

                // Wait for application startup
                bat 'timeout /t 25'

                // Show running containers
                bat 'docker ps'

                // Verify container is running
                bat 'docker inspect -f "{{.State.Running}}" test-app'
            }

            post {
                always {

                    // Cleanup container
                    bat 'docker stop test-app >nul 2>&1 || exit 0'

                    bat 'docker rm test-app >nul 2>&1 || exit 0'
                }
            }
        }
    }

    // =========================================
    // Final Pipeline Status
    // =========================================
    post {

        success {

            echo 'Pipeline SUCCESS - CI/CD completed successfully!'
        }

        failure {

            echo 'Pipeline FAILED - Check logs for details.'
        }
    }
}
