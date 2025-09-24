pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = "root"
        MYSQL_DATABASE = "studentdb"
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
                    echo "🗑️ Cleaning up old MySQL container if it exists..."
                    docker rm -f mysql-test || true

                    echo "🐬 Starting new MySQL container..."
                    docker run -d --name mysql-test \
                      -e MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD \
                      -e MYSQL_DATABASE=$MYSQL_DATABASE \
                      -p 3306:3306 \
                      mysql:8.0 --default-authentication-plugin=mysql_native_password

                    echo "⏳ Waiting for MySQL to be ready..."
                    for i in {1..30}; do
                      if docker exec mysql-test mysqladmin ping -uroot -p$MYSQL_ROOT_PASSWORD --silent; then
                        echo "✅ MySQL is up!"
                        break
                      fi
                      echo "Still waiting... ($i/30)"
                      sleep 2
                    done
                '''
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
                echo "🚀 Deploy stage here (copy JAR to server, run with java -jar, etc.)"
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
