pipeline {
  agent any
  
  stages {
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t cyberfrat:$BUILD_NUMBER .'
        }
    }
    
    stage('Test Run') {
      steps {
        sh 'docker run -d cyberfrat:$BUILD_NUMBER'
      }
    }
  } 
} 
