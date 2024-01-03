pipeline {
    agent any
    tools {
        jdk 'jdk-17'
        nodejs 'Nodejs-16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-app \
                    -Dsonar.projectKey=reddit-app'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
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
	stage("Build & Push Docker Image") {
            steps {
                script {
		     withDockerRegistry(credentialsId: 'dockerhub-credentials', toolName: 'Docker')
			 sh "docker build -t reddit-app-ci ."
                         sh "docker tag reddit-app-ci vikta96/reddit-app-ci:latest "
                         sh "docker push vikta96/reddit-app-ci:latest"
                }
            }
        }
	stage("Trivy Image Scan") {
            steps {
                sh "trivy image vikta96/reddit-app-ci:latest > trivyimage.txt" 
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
}	
