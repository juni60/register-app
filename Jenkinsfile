pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    options {
        timestamps()
    }

    environment {
        SONAR_AUTH_TOKEN = credentials('Jenkins-SonarQube-token') // must match Jenkins credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                dir('juni60/register-app') { // folder containing pom.xml
                    sh 'mvn clean verify'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('register-app') { // same folder as pom.xml
                    script {
                        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                            sh '''
                                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                                    -Dsonar.projectKey=my-project \
                                    -Dsonar.projectName="register-app-ci" \
                                    -Dsonar.host.url=http://your-sonarqube-server:9000 \
                                    -Dsonar.login=$SONAR_AUTH_TOKEN
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
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
