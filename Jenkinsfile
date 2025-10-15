pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"                 // Docker Hub
        DOCKER_CREDENTIALS_ID = "docker-hub-creds"   // Jenkins credentials
        IMAGE_NAME = "tulasigowda/fusionpact"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/tulasids5/fusionpact-devops-challenge.git'
            }
        }

        stage('Build Backend & Frontend') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Test Backend') {
            steps {
                sh 'echo "Add your backend test commands here"' 
                // e.g., python -m unittest discover backend/tests
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker tag fusionpact-devops-challenge_backend $IMAGE_NAME-backend:latest
                    docker tag fusionpact-devops-challenge_frontend $IMAGE_NAME-frontend:latest
                    docker push $IMAGE_NAME-backend:latest
                    docker push $IMAGE_NAME-frontend:latest
                    """
                }
            }
        }

        stage('Deploy to Cloud') {
            steps {
                sh 'docker-compose down'
                sh 'docker-compose up -d'
            }
        }

    }

    post {
        always {
            sh 'docker system prune -f'
        }
    }
}
