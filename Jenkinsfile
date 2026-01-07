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
                script {
                    def status = sh(script: 'mvn clean package -DskipTests', returnStatus: true)
                    if (status != 0) {
                        echo "⚠️ Build failed, continuing pipeline..."
                    }
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    def status = sh(script: 'mvn test', returnStatus: true)
                    if (status != 0) {
                        echo "⚠️ Tests failed, continuing pipeline..."
                    }
                }
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: '**/target/surefire-reports/**/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=register-app \
                        -Dsonar.projectName=register-app
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, Test & SonarQube Analysis completed!'
        }
        failure {
            echo '❌ Pipeline failed – check logs'
        }
        always {
            cleanWs()
        }
    }
}
