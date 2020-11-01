pipeline {
  environment {
    registry = "thedeepsyadav/devsecops-training"
    registryCredential = "DockerHub"
    dockerImage = ''
  }
  
  agent any
  
  stages {
    stage('Check for Secrets'){
      steps {
        sh "rm -rf trufflehog.json || true"
        sh "docker run dxa4481/trufflehog:latest --json https://github.com/deepshankaryadav/CyberFRAT-DevSecOps-Training-Sample-Flask-App.git > trufflehog.json || true"
        sh "cat trufflehog.json"
      }
    }
    
    stage('SCA'){
      steps {
        sh "pip3 install safety"
        sh "rm -rf safety.json || true"
        sh "safety check -r requirements.txt --json > safety.json || true"
        sh "cat safety.json"
      }
    }
    
    stage('SAST'){
      steps {
        sh "rm -rf bandit.json || true"
        sh "bandit -r -f=json -o=bandit.json . || true"
        sh "cat bandit.json"
      }
    }
    
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    
    stage('Push to DockerHub') {
      steps {
        script {
          docker.withRegistry('', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    
    stage('Deploy Test Application) {
      steps {
        sh 'docker stop flaskr && docker rm flaskr || true'
        sh 'docker pull thedeepsyadav/devsecops-training:latest'
        sh 'docker run -p 5000:5000 --name flaskr thedeepsyadav/devsecops-training:latest'
      }
    }

    stage('DAST Scan'){
      steps{
        sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t 'http://build.dsy.sh:5000' || true'
      }
    }
  } 
} 
