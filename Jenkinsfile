@Library('sharedLib') _
pipeline {
    agent any

    environment {
        IMAGE_TAG    = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        COMPOSE_FILE = "docker-compose.${env.BRANCH_NAME}.yml"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Docker Login') {
            steps {
                dockerLogin('dockerhub-credentials')
            }
        }
        stage('Build Images') {
            steps {
                buildImages('./backend', './frontend', IMAGE_TAG)
            }
        }
        stage('Push Images') {
            steps {
                pushImages()
            }
        }
        stage('Prepare .env') {
            steps {
                prepareEnvFile()
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'stg' || env.BRANCH_NAME == 'prod') {
                        input message: "Deploy to ${env.BRANCH_NAME}?", ok: "Deploy"
                    }
                    deployApp(COMPOSE_FILE, env.BRANCH_NAME)
                }
            }
        }
        stage('Cleanup') {
            steps {
                cleanupImages()
            }
        }
    }

    post {
        success {
            echo "✅ ${BRANCH_NAME} environment deployed successfully using Docker Hub images!"
        }
        failure {
            echo "❌ Deployment failed for ${BRANCH_NAME}. Check logs."
        }
    }
}
