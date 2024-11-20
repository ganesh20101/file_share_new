pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id'   // Jenkins Credentials ID for Docker Hub
        DOCKERHUB_REPO = 'ganesh20101/fileshare'              // Your Docker Hub repository
        FIXED_IMAGE_TAG = '24.0.03'                            // Fixed image tag
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository'
                git url: 'https://github.com/ganesh20101/file_share_new.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Get the current commit hash and branch name
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def branchName = env.BRANCH_NAME
                    
                    // Define the Docker image tag with commit hash, branch name, and fixed tag
                    def imageTag = "${DOCKERHUB_REPO}:${commitHash}"
                    def branchTag = "${DOCKERHUB_REPO}:${branchName}"
                    def fixedTag = "${DOCKERHUB_REPO}:${FIXED_IMAGE_TAG}"

                    echo "Building Docker image with tags: ${imageTag}, ${branchTag}, ${fixedTag}"

                    // Build the Docker images with --no-cache flag to avoid using cache
                    docker.build("${DOCKERHUB_REPO}:${commitHash}", "--no-cache .")
                    docker.build("${DOCKERHUB_REPO}:${branchName}", "--no-cache .")
                    docker.build("${DOCKERHUB_REPO}:${FIXED_IMAGE_TAG}", "--no-cache .")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Ensure commitHash is defined here to use in push commands
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def branchName = env.BRANCH_NAME
                    def fixedTag = "${DOCKERHUB_REPO}:${FIXED_IMAGE_TAG}"

                    echo "Pushing Docker images: ${DOCKERHUB_REPO}:${commitHash}, ${DOCKERHUB_REPO}:${branchName}, ${fixedTag}"

                    // Use Docker Pipeline plugin to push the images
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${DOCKERHUB_REPO}:${commitHash}").push()
                        docker.image("${DOCKERHUB_REPO}:${branchName}").push()
                        docker.image(fixedTag).push()
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline execution failed!"
        }
    }
}
