
pipeline {
    agent any

    environment {
        IMAGE_NAME = "social-media-strategy-fe"
        CONTAINER_NAME = "social-media-strategy-fe"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/BloomTech-Labs/social-media-strategy-fe.git'
            }
        }

        stage('Verify Node') {
            steps {
                sh 'docker --version'
                sh 'git --version'
            }
        }

        stage('Build and Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'social-api-url',
                           variable: 'API_URL'),
                    string(credentialsId: 'social-okta-domain',
                           variable: 'OKTA_DOMAIN'),
                    string(credentialsId: 'social-okta-client-id',
                           variable: 'OKTA_CLIENT_ID')
                ]) {
                    sh '''
                        docker build \
                          --build-arg REACT_APP_API_URL="$API_URL" \
                          --build-arg REACT_APP_OKTA_DOMAIN="$OKTA_DOMAIN" \
                          --build-arg REACT_APP_CLIENT_ID="$OKTA_CLIENT_ID" \
                          -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                          -t ${IMAGE_NAME}:latest .
                    '''
                }

                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p 80:80 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5
                    docker ps
                    curl -f http://localhost/ || exit 1
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

