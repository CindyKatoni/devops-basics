pipeline {
    agent any

    environment {
        DOCKER_SERVER = '54.165.158.86'
        DOCKER_USER = 'ubuntu'
        DOCKER_HUB_REPO = 'akinaregbesola/class_images'
        DOCKER_HUB_CREDENTIALS = 'dockerhub_credentials_id'
        IMAGE_TAG = 'latest'
        SSH_CREDENTIALS_ID = 'ssh-credentials-id'
        REPO_URL = 'https://github.com/theitern/devops-basics.git'
    }

    tools {
        jdk 'myjava'
        maven 'mymaven'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository..'
                withCredentials([usernamePassword(credentialsId: 'theitern', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        // Remove existing repository clone if present
                        sh "rm -rf /var/lib/jenkins/workspace/cicd-pipeline/devops-basics"
                        // Clone repository to /var/lib/jenkins/workspace/cicd-pipeline/devops-basics
                        git credentialsId: 'theitern', url: env.REPO_URL, branch: 'master', dir: '/var/lib/jenkins/workspace/cicd-pipeline/devops-basics'
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

        stage('Clone Repository Again') {
            steps {
                echo 'Cloning repository again..'
                withCredentials([usernamePassword(credentialsId: 'theitern', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        // Remove existing repository clone if present
                        sh "rm -rf /var/lib/jenkins/workspace/cicd-pipeline/devops-basics"
                        // Clone repository to /var/lib/jenkins/workspace/cicd-pipeline/devops-basics
                        git credentialsId: 'theitern', url: env.REPO_URL, branch: 'master', dir: '/var/lib/jenkins/workspace/cicd-pipeline/devops-basics'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image..'
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DOCKER_USER}@${env.DOCKER_SERVER} 'cd /var/lib/jenkins/workspace/cicd-pipeline/devops-basics && ls -la && sudo docker build -t ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG} .'
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
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
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
