pipeline {
    agent any

    environment {
        ANSIBLE_SERVER = '54.92.219.108'
        ANSIBLE_USER = 'ubuntu'
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

        stage('Copy WAR to Ansible Server') {
            steps {
                echo 'Copying WAR to Ansible Server..'
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER} 'rm -f /home/ubuntu/webapp.war'
                        scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/${env.JOB_NAME}/webapp/target/webapp.war ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER}:/home/ubuntu/
                    """
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo 'Deploying with Ansible..'
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        scp -o StrictHostKeyChecking=no -r ${env.WORKSPACE}/* ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER}:/home/ubuntu
                        ssh -o StrictHostKeyChecking=no ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER} 'ansible-playbook -i hosts playbook.yaml'
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

