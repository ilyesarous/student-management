pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "student-ci"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ilyesarous/student-management.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Start Services with Docker Compose') {
            steps {
                script {
                    sh 'docker-compose down || true'
                    sh 'docker-compose up -d --build'
                    sh 'sleep 30' // wait for MySQL to be ready
                }
            }
        }

        stage('Smoke Test') {
            steps {
                script {
                    // Check app container logs
                    sh 'docker-compose logs app || true'

                    // Example: check if app endpoint is up
                    sh 'curl -f http://localhost:8080/actuator/health || true'
                }
            }
        }

        // Optional stage: Push image to DockerHub
        // stage('Push Image') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        //             sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
        //             sh "docker-compose push"
        //         }
        //     }
        // }
    }

    post {
        always {
            echo "Cleaning up..."
            sh 'docker-compose down || true'
        }
    }
}
