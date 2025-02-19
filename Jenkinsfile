pipeline {
    agent any 
    tools {
        maven 'Maven3'
        jdk 'Java17'
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER ="praveenpeddapotula"
        DOCKER_PASS = "Dockerhub-token"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
            }
    stages{
        stage('cleanup workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from SCM'){
            steps{
                git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/praveenpeddapotula/register-app.git'
            }
        }
        stage('Build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('Test application'){
            steps{
                sh "mvn test"
            }
        }
        stage('Sonarqube Analysis'){
            steps{
               script {
                withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
               sh "mvn sonar:sonar"
                }
               }
            }
        }
        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage("Build and push docker image"){
            steps{
                script{
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')

                    }
                }
            }
        }
        stage('Trivy scan'){
            steps{
                script{
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image "${IMAGE_NAME}:${IMAGE_TAG}" --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
        stage('cleanup artifacts'){
            steps{
                script{
                    sh 'docker rmi "${IMAGE_NAME}:${IMAGE_TAG}" '
                    sh 'docker rmi "${IMAGE_NAME}:latest" '
                }
            }
        }
        stage('Trigger cd pipeline'){
            steps {
                script {
                    sh "curl -v -k --user praveen:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '3.92.136.4:8080/job/gitops-register-app-cd/buildWithParameters?token=github-cred' "
                }
            }
        }
    }
}