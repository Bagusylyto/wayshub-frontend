pipeline {
    agent any

    environment {
        APP_SERVER    = credentials('appserver')
        DIRECTORY = credentials('direktori')
        NOTIF_WEBHOOK    = credentials('notif-webhook')
    }

    stages {

        stage('pull code') {
            steps {
                sshagent(['appserver']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} << EOF
                        cd ${DIRECTORY}
                        git pull origin master
                        exit
                        EOF
                    """
                }
            }
        }

        stage('build app') {
            steps {
                sshagent(['appserver']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} << EOF
                        cd ${DIRECTORY}
                        docker compose build
                        exit
                        EOF
                    """
                }
            }
        }

        stage('push registry') {
            steps {
                sshagent(['appserver']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} << EOF
                        cd ${DIRECTORY}
                        docker compose push
                        exit
                        EOF
                    """
                }
            }
        }

        stage('deploy') {
            steps {
                sshagent(['appserver']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} << EOF
                        cd ${DIRECTORY}
                        docker compose down
                        docker compose up -d
                        exit
                        EOF
                    """
                }
            }
        }
    }

    post {
        success {
            discordSend(
                webhookURL: NOTIF_WEBHOOK,
                title: "Deploy SUCCESS",
                description: "Deploy Wayshub-Frontend Berhasil.",
                result: "SUCCESS"
            )
        }

        failure {
            discordSend(
                webhookURL: NOTIF_WEBHOOK,
                title: "Deploy FAILED",
                description: "Deploy Wayshub-Frontend Gagal.",
                result: "FAILURE"
            )
        }
    }
}
