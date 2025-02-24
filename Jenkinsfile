pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/your-app"
        DEPLOY_SERVER = "ubuntu@your-server-ip"  // Ganti dengan IP atau domain server
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'your-jenkins-credential-id', url: 'git@github.com:your-username/your-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login ke Docker Hub
                    withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u your-dockerhub-username --password-stdin"
                    }
                    // Push Docker image ke Docker Hub
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    // Login SSH ke server dan jalankan container baru
                    withCredentials([sshUserPrivateKey(credentialsId: 'server-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no $DEPLOY_SERVER << EOF
                        docker pull $DOCKER_IMAGE:latest
                        docker stop nodejs-goof || true
                        docker rm nodejs-goof || true
                        docker run -d --name nodejs-goof -p 3001:3001 $DOCKER_IMAGE:latest
                        EOF
                        '''
                    }
                }
            }
        }
    }
}
