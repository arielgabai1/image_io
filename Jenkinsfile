pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Deploy') {
            steps {
                withMaven(maven: 'Maven 3.6.2', jdk: 'OpenJDK 8') {
                    sh 'mvn clean deploy'
                }
            }
        }
    }
}
