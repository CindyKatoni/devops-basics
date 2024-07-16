pipeline {
    agent any

    environment {
        ANSIBLE_SERVER = '54.160.200.205'
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
                        sh 'rm -rf ${WORKSPACE}/devops-basics'
                        // Clone repository to ${WORKSPACE}/devops-basics
                        git credentialsId: 'theitern', url: env.REPO_URL, branch: 'master', dir: "${WORKSPACE}/devops-basics"
                    }
                }
            }
        }

        stage('Compile') {
            steps {
                dir("${WORKSPACE}/devops-basics") {
                    echo 'Compiling..'
                    sh 'mvn compile'
                }
            }
        }

        stage('Package') {
            steps {
                dir("${WORKSPACE}/devops-basics") {
                    echo 'Packaging..'
                    sh 'mvn package'
                }
            }
        }

        stage('Copy WAR to Ansible Server') {
            steps {
                echo 'Copying WAR to Ansible Server..'
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER} 'rm -f /home/ubuntu/webapp.war'
                        scp -o StrictHostKeyChecking=no ${WORKSPACE}/devops-basics/target/webapp.war ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER}:/home/ubuntu/
                    """
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo 'Deploying with Ansible..'
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        scp -o StrictHostKeyChecking=no -r ${WORKSPACE}/devops-basics/* ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER}:/home/ubuntu
                        ssh -o StrictHostKeyChecking=no ${env.ANSIBLE_USER}@${env.ANSIBLE_SERVER} 'ansible-playbook -i /home/ubuntu/hosts /home/ubuntu/playbook.yaml'
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
