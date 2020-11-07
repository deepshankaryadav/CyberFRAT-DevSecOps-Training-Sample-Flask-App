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
    
    stage('Deploy Test Application') {
      steps {
        sh 'docker stop flaskr && docker rm flaskr || true'
        sh 'docker pull thedeepsyadav/devsecops-training:$BUILD_NUMBER'
        sh 'docker run -d -p 5000:5000 --name flaskr thedeepsyadav/devsecops-training:$BUILD_NUMBER'
      }
    }

    stage('DAST Scan'){
      steps{
        sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://build.dsy.sh:5000/ || true'
      }
    }
    
    stage("Docker Security"){
      parallel{
        stage("Docker Benchmarking"){
          steps{
            sh '''
            docker run -it --net host --pid host --userns host --cap-add audit_control -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST -v /etc:/etc:ro -v /usr/bin/containerd:/usr/bin/containerd:ro -v /usr/bin/runc:/usr/bin/runc:ro -v /usr/lib/systemd:/usr/lib/systemd:ro -v /var/lib:/var/lib:ro -v /var/run/docker.sock:/var/run/docker.sock:ro --label docker_bench_security docker/docker-bench-security > dockerbench.log
            cat dockerbench.log
            '''
          }
        }
        stage("Image Scanning"){
          steps{
            sh '''
            docker run -d --name db arminc/clair-db || true
            sleep 15 # wait for db to come up
            docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan
            sleep 1
            DOCKER_GATEWAY=$(docker network inspect bridge --format "{{range .IPAM.Config}}{{.Gateway}}{{end}}")
            wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
            ./clair-scanner --ip="$DOCKER_GATEWAY" thedeepsyadav/devsecops-training:$BUILD_NUMBER || exit 0
            '''
          }
        }
      }
    } 
  } 
}
