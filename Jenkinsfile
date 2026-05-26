pipeline {
 agent any
 environment {
 // CHANGE THIS: replace 'yourusername' with your Docker Hub username
 IMAGE_NAME = 'yourusername/devops-jenkins-demo'
 IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT[0..6]}"
 LATEST_TAG = "${IMAGE_NAME}:latest"
 DOCKERHUB_CRED = 'dockerhub-credentials'
 }
 stages {
 // ─── Stage 1: Clone the repository ───────────────────
 stage('Checkout') {
 steps {
 checkout scm
 echo "Building commit: ${env.GIT_COMMIT}"
 }
 }
 // ─── Stage 2: Compile, test, package ──────────────────
 stage('Build & Test') {
 steps {
 dir('app') {
 bat 'mvn clean test'
bat 'mvn package -DskipTests -q'
 }
 }
 post {
 // Publish test results even if tests fail
 always {
 junit 'app/target/surefire-reports/*.xml'
 }
 }
 }
 // ─── Stage 3: Build Docker image ──────────────────────
 stage('Docker Build') {
 steps {
 bat "docker build -t ${IMAGE_TAG} -t ${LATEST_TAG} ."
 bat "docker images | findstr devops-jenkins-demo"
 }
 }
 // ─── Stage 4: Push to Docker Hub ──────────────────────
 stage('Docker Push') {
 steps {
 withCredentials([usernamePassword(
 credentialsId: "${DOCKERHUB_CRED}",
usernameVariable: 'DOCKER_USER',
passwordVariable: 'DOCKER_TOKEN')]) {
// Login using token via stdin (more secure)
bat 'echo %DOCKER_TOKEN% | docker login -u %DOCKER_USER% --password-stdin'

bat "docker push ${IMAGE_TAG}"
bat "docker push ${LATEST_TAG}"

bat 'docker logout'
 }
 }
 }
 // ─── Stage 5: Run container and verify health ─────────
 stage('Verify Deployment') {
 steps {
 bat "docker run -d --name test-app -p 8080:8080 ${LATEST_TAG}"

bat 'timeout /t 10'

bat 'curl http://localhost:8080/health'
 RESPONSE=$(curl -sf http://localhost:8080/health)
echo "Health check response: $RESPONSE"
echo $RESPONSE | grep -q '"status":"UP"'
echo "Container health check PASSED!"
 '''
 }
 post {
 // Always clean up the test container
 always {
 bat 'docker stop test-app'
bat 'docker rm test-app'
 }
 }
 }
 }
 post {
 success {
 echo 'Pipeline SUCCESS — Docker image pushed to Docker Hub!'
 }
 failure {
 echo 'Pipeline FAILED — check stage logs for details.'
 }
 }
}
