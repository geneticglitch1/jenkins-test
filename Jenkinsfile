pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-test"
        CONTAINER_NAME = "jenkins-test-prod"
        PORT = "3000"
        APP_DIR = "client/my-app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh "docker run --rm -v \$(pwd)/${APP_DIR}:/app -w /app node:20-alpine npm ci"
            }
        }

        stage('Test') {
            steps {
                sh "docker run --rm -v \$(pwd)/${APP_DIR}:/app -w /app node:20-alpine npm run test -- --passWithNoTests"
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d \\
                        --name ${CONTAINER_NAME} \\
                        --restart unless-stopped \\
                        -p ${PORT}:3000 \\
                        ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
    }
}