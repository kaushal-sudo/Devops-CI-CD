pipeline {
  agent any
	tools{
		maven 'Maven_Home'
	}
	
  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-hub-cred')
    REMOTE_SERVER = '172.31.9.229'
    REMOTE_USER = 'ubuntu' 	  	  
  }
	
  // Fetch code from GitHub
	
  stages {
    stage('checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/kaushal-sudo/DevOps-CI-CD-Project'

      }
    }
	  
   // Build Java application
	  
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
      }
	    
     // Post building archive Java application
	    
      post {
        success {
          archiveArtifacts artifacts: '**/target/*.jar'
        }
      }
    }
	  
  // Test Java application
	  
    stage('Maven Test') {
      steps {
        sh 'mvn test'
      }
    }
	  
   // Build docker image in Jenkins
	  
    stage('Build Docker Image') {

      steps {
        sh 'docker build -t java-app-jenkins:latest .'
        sh 'docker tag java-app-jenkins gkaushalendra/java-app-jenkins:latest'
      }
    }
	  
   // Login to DockerHub before pushing docker Image
	  
    stage('Login to DockerHub') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u    $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
	  
   // Push image to DockerHub registry
	  
    stage('Push Image to dockerHUb') {
      steps {
        sh 'docker push gkaushalendra/java-app-jenkins:latest'
      }
      post {
        always {
          sh 'docker logout'
        }
      }

    }
	  
   // Pull docker image from DockerHub and run in EC2 instance 
	  
    stage('Deploy Docker image to AWS instance') {
      steps {
        script {
          sshagent(credentials: ['awscred']) {
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker stop javaApp || true && docker rm javaApp || true'"
	  sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker pull gkaushalendra/java-app-jenkins'"
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker run --name javaApp -d -p 8081:8081 gkaushalendra/java-app-jenkins'"
		  
	
          }
        }
      }
    }
  }
}
