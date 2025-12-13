pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
        maven 'Maven 3.6.2'
    }
    parameters {
        string(name: 'RELEASE_VERSION', defaultValue: '', description: 'Version to release (e.g. 3.5). Leave empty for snapshot build.')
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
                            // If this is a Release, switch to the specific version (e.g. 3.5)
                            if (params.RELEASE_VERSION) {
                                sh "mvn versions:set -DnewVersion=${params.RELEASE_VERSION} -DgenerateBackupPoms=false"
                            }
                            // Deploy the jar (Snapshot or Release) to Artifactory
                            sh 'mvn clean deploy -s $ARTIFACTORY_SETTINGS'
                        }
                    }
                }
            }
        }

        stage('Update Git') {
            when { expression { params.RELEASE_VERSION != '' } }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'gitlab-ssh-key', keyFileVariable: 'KEY', usernameVariable: 'USER')]) {
                    script {
                        def git = "ssh -i $KEY -o StrictHostKeyChecking=no"
                        sh "git config user.email 'jenkins@example.com' && git config user.name 'Jenkins CI'"

                        // 1. Commit the Release and Tag it
                        sh "GIT_SSH_COMMAND='${git}' git commit -am 'Release ${params.RELEASE_VERSION}'"
                        sh "GIT_SSH_COMMAND='${git}' git tag -a v${params.RELEASE_VERSION} -m 'Release ${params.RELEASE_VERSION}'"
                        sh "GIT_SSH_COMMAND='${git}' git push origin v${params.RELEASE_VERSION}"

                        // 2. Auto-calculate Next Version
                        def tokens = params.RELEASE_VERSION.tokenize('.')
                        tokens[-1] = (tokens.last().toInteger() + 1).toString()
                        def nextVer = tokens.join('.') + "-SNAPSHOT"
                        echo "Next version: ${nextVer}"

                        // 3. Set Next Version, Commit, and Push Main
                        sh "mvn versions:set -DnewVersion=${nextVer} -DgenerateBackupPoms=false"
                        sh "GIT_SSH_COMMAND='${git}' git commit -am 'Prepare next ${nextVer} [skip ci]'"
                        sh "GIT_SSH_COMMAND='${git}' git push origin HEAD:main"
                    }
                }
            }
        }
    }
}
