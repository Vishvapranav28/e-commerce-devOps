
pipeline {
    agent any

    environment {
        EC2_IP = '18.60.157.137'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME ?: 'dev'}", url: 'https://github.com/Vishvapranav28/e-commerce-devOps.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = env.BRANCH_NAME == 'dev' ? 'dev' : 'prod'
                    sh "docker build -t vishvapranav/${imageTag}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    script {
                        def imageTag = env.BRANCH_NAME == 'dev' ? 'dev' : 'prod'
                        sh "echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"
                        sh "docker push $DOCKERHUB_USER/${imageTag}:latest"
                    }
                }
            }
        }

        stage('Deploy to AWS') {
            steps {
                sshagent(['ec2-ssh-key-id']) {
                    script {
                        def imageTag = env.BRANCH_NAME == 'dev' ? 'dev' : 'prod'
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                            docker pull vishvapranav/${imageTag}:latest &&
                            docker stop app || true &&
                            docker rm app || true &&
                            docker run -d -p 80:80 --name app vishvapranav/${imageTag}:latest
                        '
                        """
                    }
                }
            }
        }
    }
}
