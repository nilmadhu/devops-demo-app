pipeline {
    agent any

    environment {
        APP_NAME = "devops-demo"
        NEXUS_REGISTRY = "192.168.49.2:32001"
        NEXUS_REPO = "docker-hosted"
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${NEXUS_REGISTRY}/${NEXUS_REPO}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()   // ✅ clean everything first
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/nilmadhu/devops-demo-app.git',
                    credentialsId: 'github-devops-demo-pat'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${APP_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Docker Login to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-docker-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh 'echo $NEXUS_PASS | docker login ${NEXUS_REGISTRY} -u $NEXUS_USER --password-stdin'
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

