pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ilyesarous/student-management.git/'
            }
        }

        stage('Start MySQL for Tests') {
            steps {
                sh '''
                    docker run -d --name mysql-test \
                      -e MYSQL_ROOT_USERNAME=root \
                      -e MYSQL_ROOT_PASSWORD= \
                      -e MYSQL_DATABASE=studentdb \
                      -p 3306:3306 \
                      mysql:8.0
                '''
                sleep(time:30, unit:"SECONDS") // wait for MySQL to boot
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
                    junit 'target/surefire-reports/*.xml' // publish test results
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
                echo "Deploy stage here (e.g., copy JAR to server, run with java -jar)"
            }
        }
    }
}
