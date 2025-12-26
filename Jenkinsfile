pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"

        DOCKER_USER = "juni60x"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

        JENKINS_API_TOKEN = credentials("jenkins-api-token")
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
                withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                    sh "mvn sonar:sonar"
                }
            }
        }

        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: false,
                    credentialsId: 'jenkins-sonarqube-token'
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub') {
                        def image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        image.push()
                        image.push("latest")
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh """
                docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                --no-progress --severity HIGH,CRITICAL
                """
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                sh """
                curl -X POST \
                --user clouduser:${JENKINS_API_TOKEN} \
                'http://ec2-13-232-128-192.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token&IMAGE_TAG=${IMAGE_TAG}'
                """
            }
        }
    }

    post {
        success {
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline completed successfully.<br>Image: ${IMAGE_NAME}:${IMAGE_TAG}",
                mimeType: 'text/html',
                to: "junaidahmad.soomro01@gmail.com"
            )
        }

        failure {
            emailext(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline failed. Please check Jenkins logs.",
                mimeType: 'text/html',
                to: "junaidahmad.soomro01@gmail.com"
            )
        }
    }
}

