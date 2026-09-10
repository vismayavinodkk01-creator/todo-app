pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vismayavinodkk/two-tier-todo'
        REGISTRY_CREDENTIALS = 'dockerhub-creds'
    }

    stages {

        stage('1. Checkout SCM') {
            steps {
                checkout scm

                sh '''
                    echo "Building commit: ${GIT_COMMIT}"
                '''
            }
        }

        stage('2. Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('3. Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${REGISTRY_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "${DOCKER_PASSWORD}" | docker login \
                            -u "${DOCKER_USERNAME}" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('4. Deploy with Docker Compose') {
            steps {
                sh '''
                    docker compose down

                    docker compose pull

                    docker compose up -d
                '''
            }
        }

        stage('5. Run Database Migrations') {
            steps {
                sh '''
                    docker compose exec -T app python manage.py migrate
                '''
            }
        }

        stage('6. Verify Deployment') {
            steps {
                sh '''
                    docker compose ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }

        failure {
            echo 'Deployment failed!'
        }
    }
}
