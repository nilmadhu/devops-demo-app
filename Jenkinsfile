pipeline {
    agent any

    environment {
        APP_NAME = "devops-demo"
        NEXUS_REGISTRY = "10.104.8.101:32001" // your Nexus NodePort
        NEXUS_REPO = "docker-hosted"
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${NEXUS_REGISTRY}/${NEXUS_REPO}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t ${APP_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Login to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-docker-credentials', 
                    usernameVariable: 'NEXUS_USER', 
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                      echo $NEXUS_PASS | docker login ${NEXUS_REGISTRY} -u $NEXUS_USER --password-stdin
                    '''
                }
            }
        }

        stage('Tag & Push Image to Nexus') {
            steps {
                sh '''
                  docker tag ${APP_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
                  docker push ${FULL_IMAGE_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Image pushed successfully: ${FULL_IMAGE_NAME}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

