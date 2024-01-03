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
    }
}	
