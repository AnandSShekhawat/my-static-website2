pipeline {
    agent any
    environment {
        IMAGE_NAME = 'static-website'
        CONTAINER_NAME = 'static-website-container'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-ssh-key', 
                    url: 'https://github.com/AnandSShekhawat/my-static-website2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    if [ $(docker ps -q -f name=$CONTAINER_NAME) ]; then
                        docker stop $CONTAINER_NAME
                    fi
                    if [ $(docker ps -aq -f name=$CONTAINER_NAME) ]; then
                        docker rm $CONTAINER_NAME
                    fi
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh 'docker run -d --name $CONTAINER_NAME -p 80:80 $IMAGE_NAME'
            }
        }
    }
}
