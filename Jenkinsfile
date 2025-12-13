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
                        withCredentials([sshUserPrivateKey(credentialsId: 'gitlab-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                            script {
                                // Configure Git and SSH
                                sh """
                                    git config user.email "jenkins@example.com"
                                    git config user.name "Jenkins CI"
                                    export GIT_SSH_COMMAND="ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no"
                                """
                                
                                // 1. Set Release Version
                                sh "mvn versions:set -DnewVersion=${params.RELEASE_VERSION} -DgenerateBackupPoms=false"
                                
                                // 2. Deploy to Artifactory
                                sh 'mvn clean deploy -s $ARTIFACTORY_SETTINGS'

                                // 3. Push Release Tag
                                sh """
                                    export GIT_SSH_COMMAND="ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no"
                                    git commit -am "Release version ${params.RELEASE_VERSION}"
                                    git tag -a v${params.RELEASE_VERSION} -m "Release version ${params.RELEASE_VERSION}"
                                    git push origin v${params.RELEASE_VERSION}
                                """
                                
                                // 4. Prepare Next Snapshot
                                def nextVersion = params.NEXT_VERSION
                                if (nextVersion == '') {
                                    nextVersion = "${params.RELEASE_VERSION}-SNAPSHOT"
                                    echo "Defaulting NEXT_VERSION to ${nextVersion}"
                                }
                                
                                sh "mvn versions:set -DnewVersion=${nextVersion} -DgenerateBackupPoms=false"

                                // 5. Push Next Snapshot
                                sh """
                                    export GIT_SSH_COMMAND="ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no"
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
