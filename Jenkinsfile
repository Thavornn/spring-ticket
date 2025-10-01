pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "pinkmelon"  
        IMAGE_NAME = "pinkmelon/spring-productt" // Docker repo format
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Thavornn/17_Yin_Chheng-Thavorn_SR_SPRING_HOMEWORK001'
            }
        }

stage('Run Tests') {
    steps {
        // Run Maven tests
        sh './mvnw clean test'
    }
}


        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pat', variable: 'DOCKERHUB_TOKEN')]) {
                    sh "echo $DOCKERHUB_TOKEN | docker login -u $DOCKERHUB_USERNAME --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
