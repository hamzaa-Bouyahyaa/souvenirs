pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_1') 
        IMAGE_NAME_SERVER = 'bzabez/jenkins-exam-server' 
        IMAGE_NAME_CLIENT = 'bzabez/jenkins-exam-client' 
        APP_VERSION = '1.0.0'
    }

    stages {
        stage('Checkout') {
            steps {
                // Using checkout step instead of git for better credential handling
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/hamzaa-Bouyahyaa/souvenirs.git',
                        credentialsId: 'github_access_token'
                    ]]
                ])
            }
        }

        // stage('Run Tests') {
        //     steps {
        //         // Add error handling and proper Node.js setup
        //         nodejs(nodeJSInstallationName: 'NodeJS') {
        //             dir('server') {
        //                 sh '''
        //                     npm ci
        //                     npm test || exit 1
        //                 '''
        //             }
        //             dir('client') {
        //                 sh '''
        //                     npm ci
        //                     npm test || exit 1
        //                 '''
        //             }
        //         }
        //     }
        // }

        stage('Build Server Image') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${env.IMAGE_NAME_SERVER}:${env.APP_VERSION}")
                    }
                }
            }
        }

        stage('Build Client Image') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${env.IMAGE_NAME_CLIENT}:${env.APP_VERSION}")
                    }
                }
            }
        }

        stage('Scan Server Image') {
            steps {
                script {
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \
                        ${env.IMAGE_NAME_SERVER}:${env.APP_VERSION}
                    """
                }
            }
        }

        stage('Scan Client Image') {
            steps {
                script {
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \
                        ${env.IMAGE_NAME_CLIENT}:${env.APP_VERSION}
                    """
                }
            }
        }
        
        stage('Push Images to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_1', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
                        sh """
                            echo "\${DOCKERHUB_TOKEN}" | docker login -u "\${DOCKERHUB_USER}" --password-stdin
                            docker push ${env.IMAGE_NAME_SERVER}:${env.APP_VERSION}
                            docker push ${env.IMAGE_NAME_CLIENT}:${env.APP_VERSION}
                        """
                    }
                }
            }
        }

        // stage('Deploy') {
        //     steps {
        //         script {
        //             // Add error handling for kubectl
        //             sh """
        //                 if command -v kubectl > /dev/null; then
        //                     kubectl apply -f k8s/deployment.yaml
        //                 else
        //                     echo "kubectl not found. Please install kubectl first."
        //                     exit 1
        //                 fi
        //             """
        //         }
        //     }
        // }
    }

    // post {
    //     always {
    //         // Clean up Docker images
    //         sh """
    //             docker rmi ${env.IMAGE_NAME_SERVER}:${env.APP_VERSION} || true
    //             docker rmi ${env.IMAGE_NAME_CLIENT}:${env.APP_VERSION} || true
    //         """
    //     }
    //     success {
    //         echo "Pipeline completed successfully"
    //     }
    //     failure {
    //         echo "Pipeline failed"
    //     }
    // }
}