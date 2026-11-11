
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'social-media-strategy-fe'
        CONTAINER_NAME = 'social-media-strategy-fe'
        APP_PORT = '8081'
        API_URL = 'https://api.so-me.net/api/'
    }

    stages {
        stage('Checkout Application') {
            steps {
                dir('app') {
                    deleteDir()
                    git branch: 'master',
                        url: 'https://github.com/BloomTech-Labs/social-media-strategy-fe.git'
                }
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    docker --version
                    git --version
                    ls -la
                    ls -la app
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -f Dockerfile \
                      --build-arg REACT_APP_API_URL="$API_URL" \
                      --build-arg REACT_APP_OKTA_DOMAIN="" \
                      --build-arg REACT_APP_OKTA_CLIENT_ID="" \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest \
                      app
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p ${APP_PORT}:80 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5
                    docker ps
                    curl -f http://localhost:${APP_PORT}/
                '''
            }
        }
    }

    post {
        success {
            echo 'Frontend deployment successful!'
        }
        failure {
            echo 'Deployment failed. Check Jenkins console output.'
        }
    }
}

