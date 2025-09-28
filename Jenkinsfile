pipeline {
    agent any

    environment {
        IMAGE_REPO = "pinkmelon/springhw"          // Your Docker Hub repository
        DOCKER_CREDENTIALS_ID = "dockerhub-token" // Jenkins credential for Docker Hub
        TAG_FILE = "last-tag.txt"                 // File to track last image tag
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

                    // Default to 0 if no previous tag
                    def lastTag = 0
                    if (fileExists(TAG_FILE)) {
                        lastTag = readFile(TAG_FILE).trim().toInteger()
                    }

                    IMAGE_TAG = (lastTag + 1).toString()
                    echo "📦 New Docker image tag: ${IMAGE_TAG}"

                    // Save the tag for next build
                    writeFile file: TAG_FILE, text: IMAGE_TAG
                }
            }
        }

        // 3. Build and push Docker image (simple shell commands)
        stage('Build & Push Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image ${IMAGE_REPO}:${IMAGE_TAG}..."
                    docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .

                    echo "Pushing Docker image ${IMAGE_REPO}:${IMAGE_TAG}..."
                    docker push ${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }

    } // <-- This closes the 'stages' block

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
