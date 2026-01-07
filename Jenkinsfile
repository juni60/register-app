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
        SONAR_AUTH_TOKEN = credentials('Jenkins-SonarQube-token')
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/juni60/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                        sh """
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \
                                -Dsonar.projectKey=register-app-ci \
                                -Dsonar.projectName="Register App CI" \
                                -Dsonar.host.url=http://18.140.2.58:9000 \
                                -Dsonar.login=$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
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
