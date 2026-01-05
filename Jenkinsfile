pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java21'
        maven 'Maven3'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()  // Requires Workspace Cleanup plugin
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: ''https://github.com/juni60/register-app.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test Application') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/**/*.xml'  // Publishes test results
                }
            }
        }
    }
}
