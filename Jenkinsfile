pipeline {
    agent any
    tools {
        maven 'Maven 3.6.2'
        jdk 'openjdk 8'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Deploy') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
