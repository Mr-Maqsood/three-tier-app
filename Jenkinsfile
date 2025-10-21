@Library('sharedLib') _

pipeline {
    agent any

    environment {
        IMAGE_TAG    = "${BRANCH_NAME}-${BUILD_NUMBER}"
        COMPOSE_FILE = "docker-compose.${BRANCH_NAME}.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    checkoutSource()
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    dockerLogin('dockerhub-credentials')
                }
            }
        }

        stage('Build Images') {
            steps {
                script {
                    buildImages('./backend', './frontend', IMAGE_TAG)
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    pushImages()
                }
            }
        }

        stage('Prepare .env File') {
            steps {
                script {
                    prepareEnvFile()
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    deployApp(COMPOSE_FILE, BRANCH_NAME)
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    cleanupImages()
                }
            }
        }
    }

    post {
        success {
            echo "✅ ${BRANCH_NAME} environment deployed successfully using Docker Hub images!"
        }
        failure {
            echo "❌ Deployment failed for ${BRANCH_NAME}. Check Jenkins logs."
        }
    }
}
