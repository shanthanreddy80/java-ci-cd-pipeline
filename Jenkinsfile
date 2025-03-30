pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'http://localhost:9000'
        DOCKER_IMAGE = 'shanthanreddy80/spring-petclinic:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Commented out to prevent permission errors inside Docker/WSL environments
                    // deleteDir()
                    sh 'git clone https://github.com/spring-projects/spring-petclinic.git .'
                }
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
                    docker.image('openjdk:17-jdk').inside {
                        sh './mvnw test'
                    }
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {
                    docker.image('openjdk:17-jdk').inside {
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
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
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
