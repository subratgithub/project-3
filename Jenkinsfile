pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Docker Hub creds
        IMAGE_NAME = 'satya44jit/testing-app'
        REMOTE_HOST = 'ec2-user@35.154.15.34'
        REMOTE_APP_NAME = 'testing-app'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/chinabudhi123/project-3.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $IMAGE_NAME:latest
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ssh-access-key']) {  // <-- Updated with correct EC2 key ID
                    sh """
                        ssh -o StrictHostKeyChecking=no $REMOTE_HOST '
                            docker pull $IMAGE_NAME:latest &&
                            docker stop $REMOTE_APP_NAME || true &&
                            docker rm $REMOTE_APP_NAME || true &&
                            docker run -d --name $REMOTE_APP_NAME -p 80:80 $IMAGE_NAME:latest
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}

