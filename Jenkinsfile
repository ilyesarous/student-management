pipeline {
    agent any

    environment {
        IMAGE_NAME = "student-app"
        IMAGE_TAG = "latest"
        SONAR_PROJECT_KEY = "student-app"
        SONAR_HOST_URL = "http://10.0.1.18:9000"
        DOCKERHUB_REPO = "ilyesarous/student-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ilyesarous/student-management.git'
            }
        }

        stage('Start MySQL for Tests') {
            steps {
                script {
                    sh 'docker rm -f mysql-test || true'
                    sh '''
                        docker run -d --name mysql-test \
                          -e MYSQL_ROOT_PASSWORD=root \
                          -e MYSQL_DATABASE=studentdb \
                          -p 3306:3306 \
                          mysql:8.0 --default-authentication-plugin=mysql_native_password
                    '''
                    sh 'sleep 25'
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Run Tests (with MySQL)') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image to Docker Hub') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
            }
            steps {
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up..."
            sh 'docker rm -f mysql-test || true'
        }
    }
}
