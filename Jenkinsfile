pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
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

        stage('Verify Tools') {
            steps {
                sh '''
                    echo "===== JAVA VERSION ====="
                    java -version
                    echo "===== MAVEN VERSION ====="
                    mvn -version
                '''
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
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/**/*.xml'
                }
            }
        }
    }

    post {
        success {
            echo 'Build & Test completed successfully!'
        }
        failure {
            echo 'Build or Test failed!'
        }
        always {
            cleanWs()
        }
    }
}
