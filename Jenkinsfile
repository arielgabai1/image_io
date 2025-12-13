pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
        maven 'Maven 3.6.2'
    }
    parameters {
        string(name: 'Release_Version', defaultValue: '', description: 'Please specify a version for release (x.y.z). Leave empty for a snapshot build.')
    }
    stages {
        stage('Publish Artifact') {
            steps {
                configFileProvider([configFile(fileId: 'artifactory-settings', variable: 'ARTIFACTORY_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        script {
                            // If this is a release, switch to the specific version (x.y)
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

        stage('Update GitLab Repository') {
            when { expression { params.Release_Version != '' } }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'gitlab-ssh-key', keyFileVariable: 'KEY', usernameVariable: 'USER')]) {
                    script {
                        // We set the command in an env var for reuse in this block or repeat it
                        def gitCmd = 'ssh -i $KEY -o StrictHostKeyChecking=no'
                        sh "git config user.email 'jenkins@example.com' && git config user.name 'Jenkins CI'"

                        // Commit the Release and Tag it
                        sh """
                            export GIT_SSH_COMMAND='${gitCmd}'
                            git commit -am 'Release ${params.Release_Version} [skip ci]'
                            git tag -a v${params.Release_Version} -m 'Release ${params.Release_Version} [skip ci]'
                            git push origin v${params.Release_Version}
                        """

                        // Auto-calculate Next Version
                        def tokens = params.Release_Version.tokenize('.')
                        tokens[-1] = (tokens.last().toInteger() + 1).toString()
                        def nextVer = tokens.join('.') + "-SNAPSHOT"

                        // Set Next Version, Commit, and Push Main
                        sh "mvn versions:set -DnewVersion=${nextVer} -DgenerateBackupPoms=false"
                        sh """
                            export GIT_SSH_COMMAND='${gitCmd}'
                            git commit -am 'Bump to ${nextVer} [skip ci]'
                            git push origin HEAD:main
                        """
                    }
                }
            }
        }
    }
}