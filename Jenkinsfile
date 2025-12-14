pipeline {
    agent any
    tools {
        jdk 'OpenJDK 8'
        maven 'Maven 3.6.2'
    }
    parameters {
        string(name: 'Release_Version', defaultValue: '', description: 'Release version (x.y.z). Leave empty for a snapshot build.')
    }
    stages {
        stage('Set Release Version & Publish Artifact') {
            steps {
                configFileProvider([configFile(fileId: 'artifactory-settings', variable: 'SETTINGS')]) {
                    script {
                        // Versioning: Set release version if specified
                        if (params.Release_Version) {
                            sh "mvn versions:set -DnewVersion=${params.Release_Version} -DgenerateBackupPoms=false"
                        }
                        // Build & Deploy: 'deploy' compiles, tests, and uploads to Artifactory.
                        sh 'mvn clean deploy -s $SETTINGS'
                    }
                }
            }
        }

        stage('Update GitLab Repo') {
            when { expression { params.Release_Version != '' } }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'gitlab-ssh-key', keyFileVariable: 'KEY', usernameVariable: 'USER')]) {
                    script {
                        sh "git config user.email 'jenkins@server.com' && git config user.name 'Jenkins CI'"
                        def gitCmd = "ssh -i $KEY -o StrictHostKeyChecking=no"
                        
                        // Commit & Tag the Release
                        sh """
                            export GIT_SSH_COMMAND='${gitCmd}'
                            git commit -am 'Release ${params.Release_Version} [skip ci]'
                            git tag -a v${params.Release_Version} -m 'Release ${params.Release_Version} [skip ci]'
                            git push origin v${params.Release_Version}
                        """
                        // Calculate Next Snapshot Version
                        def nextVer = params.Release_Version.replaceFirst(/\d+$/) { (it.toInteger() + 1) } + '-SNAPSHOT'

                        // Update pom.xml & Push to Main
                        sh "mvn versions:set -DnewVersion=${nextVer} -DgenerateBackupPoms=false"
                        sh """
                            export GIT_SSH_COMMAND='${gitCmd}'
                            git commit -am 'Bump to ${nextVer}'
                            git push origin HEAD:main
                        """
                    }
                }
            }
        }
    }
}