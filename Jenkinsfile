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
      resources:
        requests:
          memory: "512Mi"
          cpu: "500m"
        limits:
          memory: "1Gi"
          cpu: "1"
    - name: jnlp
      image: jenkins/inbound-agent:latest
      args: ['$(JENKINS_SECRET)', '$(JENKINS_NAME)']
      resources:
        requests:
          memory: "256Mi"
          cpu: "200m"
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
        // Keep pods alive for debugging (10 minutes)
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: "${CD_BRANCH}", url: 'https://github.com/Thavornn/spring-ticket.git', credentialsId: 'github-token'
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
