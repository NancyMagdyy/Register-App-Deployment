pipeline {
    agent any

    tools {
        jdk "jdk8"
        maven "Maven3.9"
    }

    environment {
        APP_NAME = "nancymagdyy/register-app-deployment"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${APP_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage ('Checkout from SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/NancyMagdyy/Register-App-Deployment.git'
            }
        }

        stage ('Trivy File Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."         
            }
        }

        stage ('Application Build With Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage ('SonarQube Analysis') {
            steps {
                script {
                    sleep(time: 30, unit: 'SECONDS')
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'mvn sonar:sonar -Dsonar.ws.timeout=300'
                    }
                }
            }
        }

        stage ('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
                    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
                }
                sh '''
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage ('Scanning the Image using Trivy') {
            steps {
                sh '''
                    trivy image --timeout 5m --format table --output register-app-scan.html ${IMAGE_NAME}
                '''
            }
        }

        stage ('Push the Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
                    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
                }
                sh '''
                    docker push ${IMAGE_NAME}
                '''
            }
        }
    }
}
