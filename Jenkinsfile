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
        stage("sonar-token") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-clone-app \
                    -Dsonar.projectKey=reddit-clone-app'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-scanner'
                }
            }
        }      
    }
}	
