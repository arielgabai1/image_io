pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
        maven 'Maven 3.6.2'
    }
    parameters {
        string(name: 'RELEASE_VERSION', defaultValue: '', description: 'Version to release (e.g. 3.5). Leave empty for snapshot build.')
        string(name: 'NEXT_VERSION', defaultValue: '', description: 'Next snapshot version (e.g. 3.6-SNAPSHOT).')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Deploy') {
            when {
                expression { params.RELEASE_VERSION == '' }
            }
            steps {
                configFileProvider([configFile(fileId: 'artifactory-settings', variable: 'ARTIFACTORY_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sh 'mvn clean deploy -s $ARTIFACTORY_SETTINGS'
                    }
                }
            }
        }
        stage('Release') {
            when {
                expression { params.RELEASE_VERSION != '' }
            }
            steps {
                configFileProvider([configFile(fileId: 'artifactory-settings', variable: 'ARTIFACTORY_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sshagent(['gitlab-ssh-key']) {
                            script {
                                // Set release version
                                sh "mvn versions:set -DnewVersion=${params.RELEASE_VERSION} -DgenerateBackupPoms=false"
                                
                                // Deploy release
                                sh 'mvn clean deploy -s $ARTIFACTORY_SETTINGS'

                                // Commit and Push Release Tag
                                sh """
                                    git config user.email "jenkins@example.com"
                                    git config user.name "Jenkins CI"
                                    git commit -am "Release version ${params.RELEASE_VERSION}"
                                    git tag -a v${params.RELEASE_VERSION} -m "Release version ${params.RELEASE_VERSION}"
                                    git push origin v${params.RELEASE_VERSION}
                                """
                                
                                // Set next snapshot version
                                def nextVersion = params.NEXT_VERSION
                                if (nextVersion == '') {
                                    // Fallback: append -SNAPSHOT to the current release version if no next version is specified
                                    nextVersion = "${params.RELEASE_VERSION}-SNAPSHOT"
                                    echo "No NEXT_VERSION provided. Defaulting to ${nextVersion}"
                                }
                                sh "mvn versions:set -DnewVersion=${nextVersion} -DgenerateBackupPoms=false"

                                // Commit and Push Next Version
                                sh """
                                    git commit -am "Prepare next development iteration ${nextVersion}"
                                    git push origin HEAD:main
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}
