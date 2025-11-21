pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('code packaging'){
            steps{
                sh 'mvn package'
            }
        }
    }

    post {
        success {
            echo "✅ Build Maven réussi!"
            junit 'target/surefire-reports/*.xml'
        }
        failure {
            echo "❌ Build Maven échoué!"
        }
    }
}
