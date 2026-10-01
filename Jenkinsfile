pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install -r requirements.txt
                    ./venv/bin/pip install pytest
                    ./venv/bin/pytest
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t cicd-python-app:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['app-server-key']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@13.233.6.236 "
                            cd ~/cicd-docker-aws &&
                            git pull &&
                            docker stop cicd-python-app || true &&
                            docker rm cicd-python-app || true &&
                            docker build -t cicd-python-app . &&
                            docker run -d --name cicd-python-app -p 80:5000 cicd-python-app
                        "
                    '''
                }
            }
        }
    }
}