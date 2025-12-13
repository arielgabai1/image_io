pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
        maven 'Maven 3.6.2'
    }
    parameters {
        string(name: 'Release_Version', defaultValue: 'x.y.z', description: 'Please specify a Version for release. Leave empty for a snapshot build.')
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Publish Artifact') {
            steps {
                configFileProvider([configFile(fileId: 'artifactory-settings', variable: 'ARTIFACTORY_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        script {
                            // If this is a Release, switch to the specific version (x.y)
                            if (params.Release_Version) {
                                sh "mvn versions:set -DnewVersion=${params.Release_Version} -DgenerateBackupPoms=false"
                            }
                            // Deploy to Artifactory
                            sh 'mvn clean deploy -s $ARTIFACTORY_SETTINGS'
                        }
                    }
                }
            }
        }

        stage('Update Git') {
            when { expression { params.Release_Version != '' } }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'gitlab-ssh-key', keyFileVariable: 'KEY', usernameVariable: 'USER')]) {
                    script {
                        def git = "ssh -i $KEY -o StrictHostKeyChecking=no"
                        sh "git config user.email 'jenkins@example.com' && git config user.name 'Jenkins CI'"

                        // Commit the Release and Tag it
                        sh "GIT_SSH_COMMAND='${git}' git commit -am 'Release ${params.Release_Version}'"
                        sh "GIT_SSH_COMMAND='${git}' git tag -a v${params.Release_Version} -m 'Release ${params.Release_Version}'"
                        sh "GIT_SSH_COMMAND='${git}' git push origin v${params.Release_Version}"

                        // Auto-calculate Next Version
                        def tokens = params.Release_Version.tokenize('.')
                        tokens[-1] = (tokens.last().toInteger() + 1).toString()
                        def nextVer = tokens.join('.') + "-SNAPSHOT"
                        echo "Next version: ${nextVer}"

                        // Set Next Version, Commit, and Push Main
                        sh "mvn versions:set -DnewVersion=${nextVer} -DgenerateBackupPoms=false"
                        sh "GIT_SSH_COMMAND='${git}' git commit -am 'Prepare next ${nextVer} [skip ci]'"
                        sh "GIT_SSH_COMMAND='${git}' git push origin HEAD:main"
                    }
                }
            }
        }
    }
}