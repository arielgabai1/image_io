pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
        maven 'Maven 3.6.2'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Deploy') {
            steps {
                configFileProvider([configFile(fileId: 'artifactory-settings', variable: 'ARTIFACTORY_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sh 'mvn clean deploy -s $ARTIFACTORY_SETTINGS -Dusername=$ARTIFACTORY_USER -Dpassword=$ARTIFACTORY_PASSWORD'
                    }
                }
            }
        }
    }
}
