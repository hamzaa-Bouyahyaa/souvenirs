pipeline {
    environment {
        registryCredential = 'dockerhub_1'
        IMAGE_NAME_SERVER = 'bzabez/jenkins-exam-server'
        IMAGE_NAME_CLIENT = 'bzabez/jenkins-exam-client'
        PUSH_SUCCESS = 'false' 
    }
    agent any
    stages {
        stage('cloning Git') {
            steps {
                checkout scm
            }
        }
        stage('Building image server') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${env.IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Building image client') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${env.IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Vulnerability Scan server') {
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL ${env.IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Vulnerability Scan client') {
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL ${env.IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy Image') {
            steps {
                script {
                    try {
                        docker.withRegistry('', registryCredential) {
                            docker.image("${env.IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}").push()
                           docker.image("${env.IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}").push()
                            // Set the environment variable
                            sh "echo 'true' > .push_success"
                        }
                    } catch (Exception e) {
                        echo "Image push failed: ${e.getMessage()}"
                        sh "echo 'false' > .push_success"
                        error("Failed to push image to registry")
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
