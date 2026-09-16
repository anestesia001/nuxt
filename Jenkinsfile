def remote = [:]

pipeline {
    agent {
        label 'yandex'
    }

    parameters {
        gitParameter(
            name: 'BRANCH', 
            type: 'PT_BRANCH_TAG', 
            sortMode: 'DESCENDING_SMART', 
            selectedValue: 'TOP',
            branchFilter: 'origin/(.*)',
            quickFilterEnabled: true,
            defaultValue: 'main'
        )
    }

    environment {
        HOST = "46.243.211.91"
        PRJ_DIR = "/var/www/html"
    }

    stages {
        stage('Checkout repo') {
            steps {
                git(
                    branch: params.BRANCH,
                    url: 'git@github.com:anestesia001/nuxt.git',
                    credentialsId: 'yandex-agent'
                )
            }
        }
        stage('Configure credentials') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'jenkins-key',
                        keyFileVariable: 'PRIVATE_KEY',
                        usernameVariable: 'USERNAME'
                    )
                ]) 
                {
                    script {
                        remote.name = "${env.HOST}"
                        remote.host = "${env.HOST}"
                        remote.user = "${USERNAME}"
                        remote.identity = readFile "${PRIVATE_KEY}"
                        remote.allowAnyHosts = true
                    }
                }
            }
        }
        stage('Build static') {
            agent {
                docker {
                    image 'node'
                    label 'yandex'
                    reuseNode true
                }
            }
            steps {
                sh "npm ci --cache .npm --prefer-offline"
                sh "npm run generate"
            }
        }
        stage('Sync') {
            steps {
                sh """
                   set -e
                   tar -czvf frontend.tar.gz .output/public
                   rsync -av --delete -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" .output/public "jenkins@${env.HOST}:${env.PRJ_DIR}"
                """
                archiveArtifacts artifacts: 'frontend.tar.gz', fingerprint: true, onlyIfSuccessful: true
            }
        }
    }
}