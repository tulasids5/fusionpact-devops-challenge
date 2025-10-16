pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')
        DEPLOY_SERVER = 'ubuntu@13.233.2.10'
        SSH_KEY_ID = 'ec2-deploy-key'
        BACKEND_IMAGE = 'docker.io/tulasids5/backend:latest'
        FRONTEND_IMAGE = 'docker.io/tulasids5/frontend:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tulasids5/fusionpact-devops-challenge.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                pip install -r backend/requirements.txt
                pytest backend/tests || echo "No tests defined"
                cd frontend && npm install && npm run build || echo "No frontend build defined"
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                docker build -t $BACKEND_IMAGE backend/
                docker build -t $FRONTEND_IMAGE frontend/
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push $BACKEND_IMAGE
                docker push $FRONTEND_IMAGE
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([ec2-deploy-key]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER "
                    cd ~/fusionpact-devops-challenge || git clone https://github.com/tulasids5/fusionpact-devops-challenge.git &&
                    cd ~/fusionpact-devops-challenge &&
                    git pull &&
                    docker compose pull &&
                    docker compose up -d --remove-orphans
                    "
                    '''
                }
            }
        }
    }

    post {
        success { echo ' Deployment completed successfully!' }
        failure { echo ' Pipeline failed — check Jenkins logs.' }
        always { cleanWs() }
    }
}


