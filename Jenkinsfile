pipeline {
    agent {
        kubernetes {
            label 'docker-agent'
            defaultContainer 'docker'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: docker
      image: docker:24-dind
      securityContext:
        privileged: true
'''
        }
    }

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_CRED = 'dockerhub-token' 
        GITHUB_CRED    = 'github-token'
        DOCKER_REG     = 'pinkmelon'
        IMAGE_REPO     = "${env.DOCKER_REG}/hw-spring-product"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
        CD_BRANCH      = 'master'
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: "${CD_BRANCH}", url: 'https://github.com/Thavornn/17_Yin_Chheng-Thavorn_SR_SPRING_HOMEWORK001', credentialsId: 'github-token'
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh """
                        docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-token', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        sh """
                            echo \$DH_PASS | docker login -u \$DH_USER --password-stdin
                            docker push ${IMAGE_REPO}:${IMAGE_TAG}
                            docker tag ${IMAGE_REPO}:${IMAGE_TAG} ${IMAGE_REPO}:latest
                            docker push ${IMAGE_REPO}:latest
                            docker logout
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build & Push completed successfully!'
        }
        failure {
            echo ' Pipeline failed.'
        }
    }
}