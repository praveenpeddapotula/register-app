pipeline {
    agent any 
    tools {
        maven 'Maven3'
        jdk 'Java17'
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
    }
}