pipeline {
    tools {
        jdk 'myjava'
        maven 'mymaven'
    }
    agent any

    environment {
        DOCKER_SERVER = '54.165.158.86'
        DOCKER_USER = 'ubuntu'
        DOCKER_HUB_REPO = 'akinaregbesola/class_images'
        DOCKER_HUB_CREDENTIALS = 'dockerhub_credentials_id'
        IMAGE_TAG = 'latest'
        SSH_CREDENTIALS_ID = 'ssh-credentials-id'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning..'
                withCredentials([usernamePassword(credentialsId: 'theitern', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        git credentialsId: 'theitern', url: "https://github.com/theitern/devops-basics.git"
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
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker rm -f \$(docker ps -aq)'
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'yes | docker system prune --all'
                    """
                }
            }
        }

        stage('Copy WAR to Docker Server') {
            steps {
                echo 'Copying WAR to Docker Server..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'rm -f /home/ubuntu/webapp.war'
                        scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/${env.JOB_NAME}/webapp/target/webapp.war ${env.DOCKER_USER}@${env.DOCKER_SERVER}:/home/ubuntu/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'cd /home/ubuntu && sudo docker build -t ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG} .'
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker Image..'
                withCredentials([usernamePassword(credentialsId: env.DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'docker push ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}'
                    """
                }
            }
        }

        stage('Run Docker Image') {
            steps {
                echo 'Running Docker Image..'
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'sudo docker run -d --name our_app_container -p 8080:8080 ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}'
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
