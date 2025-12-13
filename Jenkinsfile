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
                configFile(fileId: 'artifactory-settings', variable: 'Artifactory_Settings')
                script {
                    withMaven(maven: 'Maven 3.6.2', jdk: 'OpenJDK 8', mavenSettingsFile: "${Artifactory_Settings}") {
                        sh 'mvn clean deploy'
                    }
                }
            }
        }
    }
}
