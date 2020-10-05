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
    }

}