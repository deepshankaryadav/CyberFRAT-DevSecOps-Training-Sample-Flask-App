pipeline {
    environment {
        registry = "thedeepsyadav/devsecops-training"
        registryCredential = 'DockerHub'
    }
    agent any 

    stages {
        stage('Building Docker Image') {
            steps {
                script {
                    docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Push Docker Image to DockerHub') {
            steps {
                docker.withRegistry( '', registryCredential ) { 
                    dockerImage.push() 
                }
            }
        }
        stage('Clean Up') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"  
                }
            }
        }
    }