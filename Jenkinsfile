pipeline {
    agent { label 'ec2-prod' }

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub_credentials'
        IMAGE_TAG   = "${env.BRANCH_NAME}-${BUILD_NUMBER}"
        DOCKERHUB_USER = 'maqsoodbangash'
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                    '''
                }
            }
        }

        stage('Build & Tag Images') {
            steps {
                script {
                    env.FRONTEND_TAG_DH = "${DOCKERHUB_USER}/three-tier-app-frontend:${IMAGE_TAG}"
                    env.BACKEND_TAG_DH  = "${DOCKERHUB_USER}/three-tier-app-backend:${IMAGE_TAG}"

                    sh """
                        docker build -t ${BACKEND_TAG_DH} ./backend
                        docker build -t ${FRONTEND_TAG_DH} ./frontend
                    """
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                sh """
                    docker push ${BACKEND_TAG_DH}
                    docker push ${FRONTEND_TAG_DH}
                """
            }
        }

        stage('Prepare .env for Compose') {
            steps {
                script {
                    writeFile(
                        file: '.env',
                        text: """
                        BACKEND_IMAGE=${BACKEND_TAG_DH}
                        FRONTEND_IMAGE=${FRONTEND_TAG_DH}
                        """
                    )
                }
            }
        }

        stage('Approval for Staging / Prod Deploy') {
            when {
                anyOf {
                    branch 'stg'
                    branch 'prod'
                }
            }
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        input message: "🚀 Deploy to ${env.BRANCH_NAME} environment?", ok: "Yes, Deploy"
                    }
                }
            }
        }

        stage('Deploy Environment') {
            steps {
                sh """
                    docker-compose --env-file .env down
                    docker-compose --env-file .env pull
                    docker-compose --env-file .env up -d --remove-orphans
                """
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh """
                    docker rmi ${BACKEND_TAG_DH} ${FRONTEND_TAG_DH} || true
                """
            }
        }
    }

    post {
        success {
            echo "✅ ${env.BRANCH_NAME} environment deployed successfully using Docker Hub images!"
        }
        failure {
            echo "❌ Deployment failed for ${env.BRANCH_NAME}. Check logs."
        }
    }
}
