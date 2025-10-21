@Library('sharedLib') _

pipeline {
    agent any

    environment { 
        BRANCH_NAME = 'main'
        IMAGE_TAG    = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        COMPOSE_FILE = "docker-compose.${env.BRANCH_NAME}.yml"
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
                    buildImages('./backend', './frontend', env.IMAGE_TAG)
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
                    deployApp(env.COMPOSE_FILE, env.BRANCH_NAME)
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
            echo "✅ ${env.BRANCH_NAME} environment deployed successfully using Docker Hub images!"
        }
        failure {
            echo "❌ Deployment failed for ${env.BRANCH_NAME}. Check Jenkins logs."
        }
    }
}
