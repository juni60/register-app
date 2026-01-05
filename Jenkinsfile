pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/juni60/register-app.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -B'
            }
        }

        stage('Test Application') {
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit 'target/surefire-reports/**/*.xml'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and tests completed successfully!'
        }
        failure {
            echo 'Build or tests failed!'
        }
        always {
            cleanWs()
        }
    }
}
