pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-nextjs-app"
        CONTAINER_NAME = "my-nextjs-app-prod"
        PORT = "3000"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'docker run --rm -v $(pwd):/app -w /app node:20-alpine npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm -v $(pwd):/app -w /app node:20-alpine npm run test -- --passWithNoTests'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${PORT}:3000 \
                        ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
                sh 'rm -f .env'  // don't leave .env sitting in workspace
            }
        }
    }

    post {
        failure {
            sh 'rm -f .env'  // clean up .env even if pipeline fails
        }
    }
}