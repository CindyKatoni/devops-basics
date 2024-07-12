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
                withCredentials([usernamePassword(credentialsId: 'okenyi24',
                usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        // Clone the private GitHub repository using the provided credentials
                        git credentialsId: 'okenyi24', url: "https://github.com/okenyi24/devops-basics.git"
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


