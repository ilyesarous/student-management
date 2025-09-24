pipeline {
    agent any

    tools {
        jdk 'JDK17'        // configure in Jenkins global tools
        maven 'Maven3'     // configure Maven tool there too
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ilyesarous/student-management.git/'
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
