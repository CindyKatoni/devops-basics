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

        stage('Clear Docker Server') {
            steps {
                echo 'Clearing Docker Server..'
                script {
                    sshagent(credenti
