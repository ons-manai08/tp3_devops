pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *')  // Scrutation SCM toutes les 5 minutes
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // Identifiants Docker Hub
        IMAGE_NAME_SERVER = '[onsmanai08]/mern-server'  // Nom de l'image serveur
        IMAGE_NAME_CLIENT = '[onsmanai08]/mern-client'  // Nom de l'image client
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@gitlab.com/project.git', credentialsId: 'Gitlab_ssh'
            }
        }
        stage('Build Server Image') {
            steps {
                dir('server') {
                    script {
                        // Construction de l'image du serveur
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }
        stage('Build Client Image') {
            steps {
                dir('client') {
                    script {
                        // Construction de l'image du client
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
                    }
                }
            }
        }
        stage('Scan Server Image') {
            steps {
                script {
                    // Analyse de l'image du serveur avec Trivy
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \
                        ${IMAGE_NAME_SERVER}
                    """
                }
            }
        }
        stage('Scan Client Image') {
            steps {
                script {
                    // Analyse de l'image du client avec Trivy
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \
                        ${IMAGE_NAME_CLIENT}
                    """
                }
            }
        }
        stage('Push Images to Docker Hub') {
            steps {
                script {
                    // Push des images sur Docker Hub
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        dockerImageServer.push()
                        dockerImageClient.push()
                    }
                }
            }
        }
    }
}
