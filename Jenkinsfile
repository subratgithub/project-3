pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Docker Hub creds
        IMAGE_NAME = 'techsubrat07/jenkins_dockerapp'
        REMOTE_HOST = 'ec2-user@172-31-29-178'
        REMOTE_APP_NAME = 'httpd-app'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/subratgithub/project-3.git'
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
                sshagent(['ssh_access_key']) {  // <-- Updated with correct EC2 key ID
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

