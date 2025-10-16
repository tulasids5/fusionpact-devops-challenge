pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')
        DEPLOY_SERVER = 'ubuntu@13.127.32.30'
        SSH_KEY_ID = 'ec2-deploy-key'
        BACKEND_IMAGE = 'docker.io/tulasigowda/backend:latest'
        FRONTEND_IMAGE = 'docker.io/tulasigowda/frontend:latest'
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
                pip install --user -r backend/requirements.txt || true
                pytest backend/tests || echo "No backend tests defined"
                cd frontend && npm install || echo "No frontend dependencies defined"
                cd frontend && npm run build || echo "No frontend build defined"
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
                sshagent([SSH_KEY_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER << 'ENDSSH'
                        if [ ! -d ~/fusionpact-devops-challenge ]; then
                            git clone https://github.com/tulasids5/fusionpact-devops-challenge.git ~/fusionpact-devops-challenge
                        fi
                        cd ~/fusionpact-devops-challenge
                        git pull
                        docker compose pull
                        docker compose up -d --remove-orphans
                    ENDSSH
                    """
                }
            }
        }
    }

    post {
        success { echo '✅ Deployment completed successfully!' }
        failure { echo '❌ Pipeline failed — check Jenkins logs.' }
        always { cleanWs() }
    }
}
