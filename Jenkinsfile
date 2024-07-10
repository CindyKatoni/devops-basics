pipeline {
    tools {
        jdk 'myjava'
        maven 'mymaven'
    }
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'cloning..'
                // Use withCredentials to provide GitHub credentials
                withCredentials([usernamePassword(credentialsId: 'theitern', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        // Clone the private GitHub repository using the provided credentials
                        git credentialsId: 'theitern', url: "https://github.com/theitern/devops-basics.git"
                    }
                }
            }
        }

        stage('Compile') {
            steps {
                echo 'compiling..'
                sh 'mvn compile'
            }
        }

        stage('CodeReview') {
            steps {
                echo 'codeReview'
                sh 'mvn pmd:pmd'
            }
        }

    stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
    }
}

