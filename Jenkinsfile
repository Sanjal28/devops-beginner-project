pipeline {
    agent any

    environment {
        JFROG_REGISTRY = "trial0dpg9z.jfrog.io"
        JFROG_REPO     = "docker-trial"
        IMAGE_NAME    = "devops-frontend"
        IMAGE_TAG     = "v${BUILD_NUMBER}"
        FULL_IMAGE    = "${JFROG_REGISTRY}/${JFROG_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:ci .
                """
            }
        }

        stage('Login to JFrog') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-docker',
                    usernameVariable: 'JFROG_USER',
                    passwordVariable: 'JFROG_TOKEN'
                )]) {
                    sh """
                        echo "\$JFROG_TOKEN" | docker login ${JFROG_REGISTRY} \
                        -u "\$JFROG_USER" --password-stdin
                    """
                }
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:ci ${FULL_IMAGE}
                    docker push ${FULL_IMAGE}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    sed "s|IMAGE_PLACEHOLDER|${FULL_IMAGE}|g" deployment.yaml \
                    | kubectl apply -f -
                """
            }
        }
    }

    post {
        success {
            echo "✅ Successfully deployed ${FULL_IMAGE} to Kubernetes"
        }
    }
}

