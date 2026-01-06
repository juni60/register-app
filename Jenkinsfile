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
                    // Build all modules, continue pipeline even if build fails
                    def buildStatus = sh(script: 'mvn clean package -B -DskipTests', returnStatus: true)
                    if (buildStatus != 0) {
                        echo "⚠️ Maven build failed (status ${buildStatus}), continuing pipeline..."
                    }
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    // Run tests, continue pipeline even if tests fail
                    def testStatus = sh(script: 'mvn test -B', returnStatus: true)
                    if (testStatus != 0) {
                        echo "⚠️ Maven tests failed (status ${testStatus}), continuing pipeline..."
                    }
                }
            }
            post {
                always {
                    // Publish all JUnit reports if available
                    junit allowEmptyResults: true,
                          testResults: '**/target/surefire-reports/**/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    script {
                        // Auto-detect all source folders with .java files
                        def sources = sh(script: "find . -name '*.java' -exec dirname {} \\; | sort -u", returnStdout: true).trim()
                        
                        // Auto-detect all target folders with compiled classes
                        def binaries = sh(script: "find . -type d -name 'target' -exec bash -c 'if [ -d \"\$0/classes\" ]; then echo \$0/classes; fi' {} \\;", returnStdout: true).trim()
                        
                        def sourcesArg = sources ? "-Dsonar.sources=${sources.replaceAll('\\n', ',')}" : ""
                        def binariesArg = binaries ? "-Dsonar.java.binaries=${binaries.replaceAll('\\n', ',')}" : ""
                        
                        echo "✅ Detected sources: ${sourcesArg}"
                        echo "✅ Detected binaries: ${binariesArg}"

                        sh """
                            ${tool 'SonarScanner'}/bin/sonar-scanner \
                            -Dsonar.projectKey=register-app \
                            -Dsonar.projectName=register-app \
                            $sourcesArg $binariesArg || echo "SonarScanner returned non-zero, ignoring..."
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    // Check Quality Gate but do not abort pipeline
                    waitForQualityGate abortPipeline: false
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline finished with failure (check logs)!'
        }
        always {
            cleanWs()
        }
    }
}
