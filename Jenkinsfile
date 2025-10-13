pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Your Jenkins DockerHub credentials ID
        IMAGE_NAME = "arunchintu-1/python-ci-cd"                 // Replace with your Docker Hub username/repo
    }

    stages {

        stage('Build') {
            steps {
                echo "=== Creating Virtual Environment & Installing Dependencies ==="
                sh '''
                    apt-get update -y
                    apt-get install -y python3-venv python3-pip
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "=== Building Docker Image ==="
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                echo "=== Logging into Docker Hub & Pushing Image ==="
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "=== Deploying Docker Container ==="
                sh '''
                    docker stop python-app || true
                    docker rm python-app || true
                    docker run -d -p 5000:5000 --name python-app $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
