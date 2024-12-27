pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'vietpham18/server_golang'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/hylasxl/devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy Golang to DEV') {
            steps {
                echo 'Deploying to DEV...'
                try {
                    sh 'docker image pull vietpham18/server_golang:latest'
                    sh 'docker container stop golang-jenkins || echo "this container does not exist"'
                    sh 'docker network create dev || echo "this network exists"'
                    sh 'echo y | docker container prune'
                    sh 'docker container run -d --rm --name server_golang -p 4000:4000 --network dev vietpham18/server_golang:latest'

                    sh '''
                        curl -s -X POST https://api.telegram.org/bot7686490744:AAF3MWixwEm0e6SJZu520Uu8pNmYNB2q7VU/sendMessage \
                        -d chat_id=-4696233151 \
                        -d text="Deployment to DEV successful!"
                    '''
                } catch (Exception e) {
                    sh '''
                        curl -s -X POST https://api.telegram.org/bot7686490744:AAF3MWixwEm0e6SJZu520Uu8pNmYNB2q7VU/sendMessage \
                        -d chat_id=-4696233151 \
                        -d text="Deployment to DEV failed: ${e.message}"
                    '''
                    throw e
                }
            }
        }

    }

    post {
        always {
            cleanWs()
        }
    }
}