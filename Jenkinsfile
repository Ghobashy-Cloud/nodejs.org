pipeline {
    agent any

    stages {
        stage("Checkout code") {
            steps {
                script {
                    if (!fileExists('nodejs.org')) {
                        sh 'git clone https://github.com/Ghobashy-Cloud/nodejs.org'
                    }
                    dir('nodejs.org') {
                        sh 'git fetch origin'
                        sh 'git checkout main'
                        sh 'git pull'
                    }
                }
            }
        }
        stage("Install dependencies") {
            steps {
                dir('nodejs.org') {
                    sh 'npm ci'
                }
            }
        }
        stage("Run unit testing") {
            steps {
                dir('nodejs.org') {
                    sh 'npm run test'
                }
            }
        }
        stage("Dockerize") {
            steps {
                dir('nodejs.org') {
                    sh 'docker build -t khaled55/nodejs.org .'
                }
            }
        }
        stage("Push Docker image") {
            steps {
                dir('nodejs.org') {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push khaled55/nodejs.org'
                    }
                }
            }
        }
    }
}
