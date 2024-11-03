pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') 
        IMAGE_NAME_SERVER = 'bzabez/server' 
        IMAGE_NAME_CLIENT = 'bzabez/client' 
        APP_VERSION = '1.0.0' // Example versioning
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', // Change this to your main branch if different
                    url: 'git@github.com:hamzaa-Bouyahyaa/souvenirs.git', // Update with your GitHub repository URL
                    credentialsId: 'github_jenkins_key' // Ensure this ID matches your GitHub credentials in Jenkins
            }
        }

        stage('Run Tests') {
            steps {
                dir('server') {
                    sh 'npm install && npm test'
                }
                dir('client') {
                    sh 'npm install && npm test'
                }
            }
        }

        stage('Build Server Image') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}:${APP_VERSION}")
                    }
                }
            }
        }

        stage('Build Client Image') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}:${APP_VERSION}")
                    }
                }
            }
        }

        stage('Scan Server Image') {
            steps {
                script {
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                        aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \\
                        ${IMAGE_NAME_SERVER}
                    """
                }
            }
        }

        stage('Scan Client Image') {
            steps {
                script {
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                        aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL \\
                        ${IMAGE_NAME_CLIENT}
                    """
                }
            }
        }
        
        stage('Push Images to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
                        sh '''
                            echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
                            docker push ${IMAGE_NAME_SERVER}
                            docker push ${IMAGE_NAME_CLIENT}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Add your deployment logic here
                    sh "kubectl apply -f k8s/deployment.yaml" // Example for Kubernetes
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
            // Example: Send email or Slack notification
        }
        failure {
            echo "Pipeline failed"
            // Example: Send email or Slack notification
        }
    }
}
