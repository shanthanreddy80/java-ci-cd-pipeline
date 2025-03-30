pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'http://sonarqube:9000'
        DOCKER_IMAGE = 'shanthanreddy80/spring-petclinic:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.image('openjdk:17-jdk').inside {
                        sh './mvnw clean package'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image('openjdk:11-jdk').inside {
                        sh './mvnw test'
                    }
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {
                    docker.image('openjdk:8-jdk').inside {
                        sh "./mvnw sonar:sonar -Dsonar.host.url=${SONARQUBE_SERVER}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "docker login -u shanthanreddy80 -p YOUR_PASSWORD"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}

