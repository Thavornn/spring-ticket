pipeline {
    // Use a Docker image with Docker CLI installed
    agent {
        docker {
            image 'docker:24.0-dind'  // Docker-in-Docker image
            args '--privileged'        // Needed for DinD
        }
    }

    environment {
        IMAGE_REPO = "pinkmelon/springhw" // Your Docker Hub repository
        TAG_FILE = "last-tag.txt"         // File to track last image tag
    }

    stages {

        // 1. Checkout the Spring project
        stage('Checkout Project') {
            steps {
                echo "Step 1: Checking out the project from Git..."
                checkout scm
            }
        }

        // 2. Determine next Docker image tag
        stage('Determine Image Tag') {
            steps {
                script {
                    echo "Step 2: Determining the next image tag..."

                    def lastTag = 0
                    if (fileExists(TAG_FILE)) {
                        lastTag = readFile(TAG_FILE).trim().toInteger()
                    }

                    // Use def to avoid warning
                    def imageTag = (lastTag + 1).toString()
                    env.IMAGE_TAG = imageTag
                    echo "📦 New Docker image tag: ${IMAGE_TAG}"

                    // Save the tag for next build
                    writeFile file: TAG_FILE, text: IMAGE_TAG
                }
            }
        }

        // 3. Build and push Docker image
        stage('Build & Push Docker Image') {
            steps {
                echo "Step 3: Building and pushing Docker image..."
                sh '''
                    docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .
                    docker push ${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }
    }

    // 4. Post actions
    post {
        success {
            echo "🎉 Pipeline finished! Docker image: ${IMAGE_REPO}:${IMAGE_TAG} pushed successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check Jenkins logs for details."
        }
    }
}
