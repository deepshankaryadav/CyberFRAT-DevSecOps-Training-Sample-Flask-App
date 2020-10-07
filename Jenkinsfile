pipeline {
    environment {
        registry = "thedeepsyadav/devsecops-training"
        registryCredential = 'DockerHub'
        dockerImage = ''
    }
    agent any 

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push('staging') 
                    }
                }
            }
        }

        stage('Server Clean Up') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"  
                }
            }

        stage('Download New Build') {
            steps {
                sshagent(['AppSec']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@159.65.157.103 "uptime && docker pull $registry:staging && docker stop devsecops-flaskr && docker rm devsecops-flaskr"'
                    }
                }
            }
        
        stage('Deploy New Build') {
            steps {
                sshagent(['AppSec']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@159.65.157.103 "docker run -d --name devsecops-flaskr -p 127.0.0.1:5000:5000 $registry:staging"'
                    }
                }
            }
        
        stage('Prisma Cloud Scan Scan') {
            steps {
                prismaCloudScanImage ca: '',
                cert: '',
                dockerAddress: 'unix:///var/run/docker.sock',
                image: 'thedeepsyadav/devsecops-training:*',
                key: '',
                logLevel: 'debug',
                podmanPath: '',
                project: '',
                resultsFile: 'prisma-cloud-scan-results.json',
                sh 'cat prisma-cloud-scan-results.json'
                ignoreImageBuildTime:true,
            }
        }
    }
}