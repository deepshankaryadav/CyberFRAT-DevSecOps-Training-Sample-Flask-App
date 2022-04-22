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
        sh "pip3 install -r requirements.txt"
        sh "rm -rf trufflehog.json || true"
        sh "docker run dxa4481/trufflehog:latest --json https://github.com/deepshankaryadav/CyberFRAT-DevSecOps-Training-Sample-Flask-App.git > trufflehog.json || true"
        sh "cat trufflehog.json"
      }
    }

    stage('Snyk SCA Scan'){
      tools {
        snyk 'Snyk'
      }
      steps {
        snykSecurity(
          snykInstallation: 'Snyk',
          snykTokenId: 'SnykToken',
          failOnIssues: 'false',
          additionalArguments: '--command=python3'
        )
      }
    }
    
    stage('SAST'){
      steps {
        sh "rm -rf bandit.json || true"
        sh "bandit -r -f=json -o=bandit.json . || true"
        sh "cat bandit.json || true"
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
            dockerImage.push('latest')
          }
        }
      }
    }
    
    stage('Deploy Test Application') {
      steps {
        sh 'docker stop flaskr && docker rm flaskr || true'
        sh 'docker pull thedeepsyadav/devsecops-training:latest'
        sh 'docker run -d -p 5000:5000 --name flaskr thedeepsyadav/devsecops-training:latest'
      }
    }
    
    stage("Docker Security"){
      parallel{
        stage("Docker Benchmarking"){
          steps{
            sh '''
            docker pull docker/docker-bench-security
            docker run --net host --pid host --cap-add audit_control -v /var/lib:/var/lib -v /var/run/docker.sock:/var/run/docker.sock -v /usr/lib/systemd:/usr/lib/systemd -v /etc:/etc --label docker_bench_security docker/docker-bench-security || true
            '''
          }
        }
        
        stage('Prisma Cloud Scan') {
          steps {
            // Scan the image
            prismaCloudScanImage ca: '',
            cert: '',
            dockerAddress: 'unix:///var/run/docker.sock',
            image: 'thedeepsyadav/devsecops-training',
            key: '',
            logLevel: 'debug',
            podmanPath: '',
            project: '',
            resultsFile: 'prisma-cloud-scan-results.json',
            ignoreImageBuildTime:true
          }
        }
      }
    }
    
    stage("Deploy to PROD"){
      steps{
        script{

          input message: 'Do you want to deploy in production?', ok: "OK"

        }
      }
    }
  } 
}
