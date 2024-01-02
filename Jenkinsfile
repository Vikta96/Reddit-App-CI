pipeline {
    agent any
    tools {
        jdk 'Jdk17'
        nodejs 'Node16'
    }
    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Vikta96/reddit-clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-clone-ci \
                    -Dsonar.projectKey=reddit-clone-ci'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
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
			  sh "docker build -t reddit-clone ."
                          sh "docker tag reddit-clone vikta96/reddit-clone:latest "
                          sh "docker push vikta96/reddit-clone:latest
                     }
                 }
             }
         }
	 stage("Trivy Image Scan") {
             steps {
                 sh "trivy image vikta96/reddit-clone:latest > trivyimage.txt" 
                 }
             }
         }
	 stage ('Cleanup Artifacts') {
             steps {
                 script {
                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
         }
	 stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-65-2-187-142.ap-south-1.compute.amazonaws.com:8080/job/Reddit-Clone-CD/buildWithParameters?token=gitops-token'"
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