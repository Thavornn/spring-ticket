pipeline {
    agent {
        kubernetes {
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
      command:
        - cat
      tty: true
    - name: git
      image: alpine/git
      command:
        - cat
      tty: true
'''
        }
    }

    environment {
        DOCKER_REG = 'pinkmelon'
        IMAGE_REPO = "${env.DOCKER_REG}/spring-ticket"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        CD_BRANCH  = 'main'
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {
        stage('Checkout Source') {
            steps {
                container('git') {
                    sh """
                        git clone -b ${CD_BRANCH} https://github.com/Thavornn/spring-ticket.git .
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh """
                        docker version
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
            echo '❌ Pipeline failed.'
        }
    }
}
