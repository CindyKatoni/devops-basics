pipeline {
    tools {
        jdk 'myjava'
        maven 'mymaven'
    }
    agent any

    environment {
        DOCKER_SERVER = '18.234.62.234'
        DOCKER_USER = 'ubuntu'
        DOCKER_HUB_REPO = 'akinaregbesola/private'
        DOCKER_HUB_CREDENTIALS = 'dockerhub-credentials-id'
        IMAGE_TAG = 'latest'
        SSH_CREDENTIALS_ID = 'SSH_CREDENTIALS_ID'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning..'
                withCredentials([usernamePassword(credentialsId: 'theitern', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        git credentialsId: 'theitern', url: "https://github.com/theitern/DevopsBasics.git"
                    }
                }
            }
        }

        stage('Compile') {
            steps {
                echo 'Compiling..'
                sh 'mvn compile'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging..'
                sh 'mvn package'
            }
        }

        stage('Clear Docker Server') {
            steps {
                echo 'Clearing Docker Server..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    script {
                        // Remove all running containers
                        sh "ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker rm -f \$(docker ps -aq)'"

                        // Automatically confirm "yes" for Docker system prune
                        sh "yes | ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker system prune --all'"
                    }
                }
            }
        }

        stage('Copy WAR to Docker Server') {
            steps {
                echo 'Copying WAR to Docker Server..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'rm -f /home/ubuntu/webapp.war'
                        scp /var/lib/jenkins/workspace/${env.JOB_NAME}/webapp/target/webapp.war ${env.DOCKER_USER}@${env.DOCKER_SERVER}:/home/ubuntu/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'cd /home/ubuntu && sudo docker build -t ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG} .'
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker Image..'
                withCredentials([usernamePassword(credentialsId: env.DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        ssh ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        ssh ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker push ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}'
                    """
                }
            }
        }

        stage('Run Docker Image') {
            steps {
                echo 'Running Docker Image..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'sudo docker run -d --name our_app_container -p 8080:8080 ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}'
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
