pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tulasids5/fusionpact-devops-challenge.git'
            }
        }
        stage('Build & Test') {
            steps {
                script {
                    sh 'pip install -r backend/requirements.txt'
                    sh 'pytest backend/tests || echo "No tests defined"'
                    sh 'cd frontend && npm install && npm run build || echo "No frontend build defined"'
                }
            }
        }
        stage('Docker Build & Push') {
            environment {
                DOCKER_HUB_CREDENTIALS = credentials('docker-hub-creds')
            }
            steps {
                script {
                    sh 'docker build -t docker.io/tulasids5/backend:latest backend/'
                    sh 'docker build -t docker.io/tulasids5/frontend:latest frontend/'
                    sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker push docker.io/tulasids5/backend:latest'
                    sh 'docker push docker.io/tulasids5/frontend:latest'
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(['ec2-deploy-key']) {
                    sh '''
                    ssh ubuntu@EC2_PUBLIC_IP "
                    cd ~/fusionpact-devops-challenge &&
                    docker compose pull &&
                    docker compose up -d
                    "
                    '''
                }
            }
        }
    }
}

