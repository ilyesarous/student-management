pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = ""
        MYSQL_DATABASE = "studentdb"
        MYSQL_USER = "root"
        MYSQL_PASSWORD = ""
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ilyesarous/student-management.git/'
            }
        }

        stage('Start MySQL for Tests') {
            steps {
                sh '''
                    docker rm -f mysql-test || true
                    docker run -d --name mysql-test \
                      -e MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD \
                      -e MYSQL_DATABASE=$MYSQL_DATABASE \
                      -e MYSQL_USER=$MYSQL_USER \
                      -e MYSQL_PASSWORD=$MYSQL_PASSWORD \
                      -p 3306:3306 \
                      mysql:8.0 \
                      --default-authentication-plugin=mysql_native_password
                '''
                echo "⏳ Waiting for MySQL to be ready..."
                sleep(time:30, unit:"SECONDS")
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploy stage here (e.g., copy JAR to server, run with java -jar)"
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up MySQL container"
            sh 'docker rm -f mysql-test || true'
        }
    }
}
