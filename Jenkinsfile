pipeline {
    agent any

    environment {
        IMAGE_NAME = "student-app"
        IMAGE_TAG = "latest"
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
                    // Stop old MySQL container if exists
                    sh 'docker rm -f mysql-test || true'

                    // Run fresh MySQL container
                    sh '''
                        docker run -d --name mysql-test \
                          -e MYSQL_ROOT_PASSWORD=root \
                          -e MYSQL_DATABASE=studentdb \
                          -p 3306:3306 \
                          mysql:8.0 --default-authentication-plugin=mysql_native_password
                    '''
                    // wait for DB to boot
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

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
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
