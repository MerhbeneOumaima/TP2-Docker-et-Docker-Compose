pipeline {
    agent any
    triggers {
        pollSCM ('H/5 * * * *')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME_SERVER = 'merhebenoumaima/mern-server'
        IMAGE_NAME_CLIENT = 'merhebenoumaima/mern-client'
    }
    stages {
        stage ('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:MerhbeneOumaima/TP2-Docker-et-Docker-Compose.git', credentialsId: 'Gitlab_ssh'
            }
        }

        stage ('Check Changes for Server') {
            steps {
                script {
                    // Vérifiez si des modifications ont eu lieu dans le dossier 'server'
                    def serverChanged = sh(script: "git diff --name-only HEAD~1 HEAD | grep '^server/'", returnStdout: true).trim()
                    if (serverChanged) {
                        currentBuild.description = "Changes detected in server"
                        env.SERVER_CHANGED = 'true'
                    } else {
                        env.SERVER_CHANGED = 'false'
                    }
                }
            }
        }

        stage ('Check Changes for Client') {
            steps {
                script {
                    // Vérifiez si des modifications ont eu lieu dans le dossier 'client'
                    def clientChanged = sh(script: "git diff --name-only HEAD~1 HEAD | grep '^client/'", returnStdout: true).trim()
                    if (clientChanged) {
                        currentBuild.description = "Changes detected in client"
                        env.CLIENT_CHANGED = 'true'
                    } else {
                        env.CLIENT_CHANGED = 'false'
                    }
                }
            }
        }

        stage ('Build Server Image') {
            when {
                expression { return env.SERVER_CHANGED == 'true' }
            }
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }

        stage ('Build Client Image') {
            when {
                expression { return env.CLIENT_CHANGED == 'true' }
            }
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
                    }
                }
            }
        }

        stage ('Scan Server Image') {
            when {
                expression { return env.SERVER_CHANGED == 'true' }
            }
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \
                    ${IMAGE_NAME_SERVER}
                    """
                }
            }
        }

        stage ('Scan Client Image') {
            when {
                expression { return env.CLIENT_CHANGED == 'true' }
            }
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \
                    ${IMAGE_NAME_CLIENT}
                    """
                }
            }
        }

        stage ('Push Images to Docker Hub') {
            when {
                anyOf {
                    expression { return env.SERVER_CHANGED == 'true' }
                    expression { return env.CLIENT_CHANGED == 'true' }
                }
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        if (env.SERVER_CHANGED == 'true') {
                            dockerImageServer.push()
                        }
                        if (env.CLIENT_CHANGED == 'true') {
                            dockerImageClient.push()
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Nettoyer les images et les conteneurs temporaires
                echo "Cleaning up Docker images and containers..."
                sh 'docker system prune -af'  // Supprimer toutes les images inutilisées, les conteneurs arrêtés et les volumes non utilisés
            }
        }
    }
}
