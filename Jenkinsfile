pipeline {
    agent any

    environment {
        IMAGE_NAME = 'sentiment-ai'
        REGISTRY = 'ghcr.io/VOTRE_PSEUDO'
        IMAGE_TAG = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log --oneline -5'
            }
        }

        stage('Lint') {
            steps {
                sh '''
                docker run --rm \
                -v $(pwd):/app \
                -w /app \
                python:3.11-slim \
                sh -c "pip install flake8 -q && flake8 src/"
                '''
            }
        }

        stage('Build & Test') {
            steps {

                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."

                sh '''
                docker run --rm \
                ${IMAGE_NAME}:${IMAGE_TAG} \
                pytest tests/ -v
                '''
            }
        }

        stage('Push') {

            when {
                branch 'main'
            }

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'github-token',
                    usernameVariable: 'REGISTRY_USER',
                    passwordVariable: 'REGISTRY_PASS'
                )]) {

                    sh '''
                    echo $REGISTRY_PASS | docker login ghcr.io \
                    -u $REGISTRY_USER --password-stdin

                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }

            }
        }

    }

    post {
        always {
            sh 'docker compose down -v || true'
        }
    }
}