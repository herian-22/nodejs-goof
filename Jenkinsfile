pipeline {
    agent any

    environment {
        // Menetapkan nama image Docker
        DOCKER_IMAGE = "herian-22/nodejs-goof"  // Ganti dengan nama image Docker yang sesuai
        DEPLOY_SERVER = "deploy@192.168.9.117"  // Ganti dengan IP atau domain server Anda
    }

    stages {
        stage('Checkout') {
            steps {
                // Menarik kode dari GitHub menggunakan kredensial SSH
                git branch: 'main', credentialsId: 'GitHub-SSH-Key', url: 'git@github.com:herian-22/nodejs-goof.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Membangun Docker image dari Dockerfile yang ada di repo
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login ke Docker Hub menggunakan kredensial yang disimpan di Jenkins
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
                    // Menggunakan SSH untuk login ke server dan jalankan container baru dengan image yang sudah dipush
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
