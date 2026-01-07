pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    // Explicitly call the Sonar Maven plugin without relying on pom.xml
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed – check logs'
        }
        success {
            echo '✅ Pipeline succeeded'
        }
        always {
            cleanWs()
        }
    }
}
