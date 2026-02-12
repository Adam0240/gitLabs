pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Branch to build')
        string(name: 'GIT_REPO', defaultValue: 'git@github.com:Adam0240/CS410.git', description: 'Git repo SSH URL')
        string(name: 'CREDENTIALS', defaultValue: 'ssh_key', description: 'Jenkins Credentials ID')
    }

    environment {
        // Change this if your assignment expects a specific repo/image name
        IMAGE_NAME = "textadventure-ci"
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: "${params.GIT_BRANCH}",
                    url: "${params.GIT_REPO}",
                    credentialsId: "${params.CREDENTIALS}"
            }
        }

        stage('Determine Image Tag and Build Image') {
            steps {
                sh '''
                    set -e

                    echo "Docker version:"
                    docker --version

                    echo "Docker info (may fail if daemon not running):"
                    docker info || true

                    # Make a Docker-safe tag from branch name
                    SAFE_BRANCH=$(echo "${GIT_BRANCH}" | tr '/ ' '--' | tr -cd 'A-Za-z0-9_.-')

                    # Get short commit hash from checked-out repo
                    COMMIT=$(git rev-parse --short HEAD)

                    IMAGE_TAG="${SAFE_BRANCH}-${COMMIT}-${BUILD_NUMBER}"

                    echo "Building image: ${IMAGE_NAME}:${IMAGE_TAG}"

                    # Build context is the workspace root (where your Dockerfile should be)
                    docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .

                    echo "Built: ${IMAGE_NAME}:${IMAGE_TAG}"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
