pipeline {
    agent any
    tools {
        jdk 'reddit-jdk'
        nodejs 'reddit-nodejs'
    }
    environment {
        SCANNER_HOME = tool 'SonarQubeScanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/Vikta96/Reddit-App-CI.git'
            }
        }
        stage("sonar-token") {
            steps {
                withSonarQubeEnv('SonarQubeScanner') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-sonar-project \
                    -Dsonar.projectKey=reddit-sonar-project'''
                }
            }
        } 
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'reddit sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub-credentials', toolName: 'reddit-docker') {   
                        sh "docker build -t reddit-app-ci ."
                        sh "docker tag reddit-app-ci vikta96/reddit-app-ci "
                        sh "docker push vikta96/reddit-app-ci:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                script {
                    sh "trivy image vikta96/reddit-app-ci:latest > trivyimage.txt" 
                }
            }
        }       
    
    }
    post {
        always {
           emailext attachLog: true,
               subject: "'${currentBuild.result}'",
               body: "Project: ${env.JOB_NAME}<br/>" +
                   "Build Number: ${env.BUILD_NUMBER}<br/>" +
                   "URL: ${env.BUILD_URL}<br/>",
               to: 'anandbm1995@gmail.com',                              
               attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
     }
    
}
