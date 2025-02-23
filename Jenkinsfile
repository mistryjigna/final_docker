pipeline {
  agent any
	
  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-hub-cred')
    REMOTE_SERVER = '52.73.28.146'
    REMOTE_USER = 'ec2-user' 	  	  
  }
	
  // Fetch code from GitHub
	
  stages {
    stage('checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/mistryjigna/final_docker'

      }
    }
	  
   // Build Java application
//tools { 
//        maven 'Maven 3.9.2' 
//    }
   // stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
       // }
    //}
//
	  
	  
   // stage('Maven 3.9.2') {
     // steps {
       // sh 'mvn clean install'
      //}
	    
     // Post building archive Java application
	    
      post {
        success {
          archiveArtifacts artifacts: '**/*.jar'
        }
      }
    }
	  
  // Test Java application
	  
    stage('M2_HOME Test') {
      steps {
        sh 'mvn test'
      }
    }
	  
   // Build docker image in Jenkins
	  
    stage('Build Docker Image') {

      steps {
        sh 'docker build -t javawebapp:latest .'
        sh 'docker tag javawebapp palakbhawsar/javawebapp:latest'
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
        sh 'docker push palakbhawsar/javawebapp:latest'
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
	  sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker pull palakbhawsar/javawebapp'"
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker run --name javaApp -d -p 8081:8081 palakbhawsar/javawebapp'"
          }
        }
      }
    }
  }
}
