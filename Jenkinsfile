pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')   // Jenkins DockerHub credentials ID
        IMAGE_NAME = "arunchintu-1/python-ci-cd"                  // Replace with your Docker Hub username/repo
    }

    stages {
        stage('Build') {
            steps {
                echo "=== Installing Python dependencies ==="
                // Simplified: no virtual environment needed
                sh '''
                    python3 --version
                    pip3 install --upgrade pip
                    pip3 install -r requirements.txt
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "=== Building Docker image ==="
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                echo "=== Logging into Docker Hub and pushing image ==="
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "=== Deploying container ==="
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
            echo "❌ Pipeline failed. Please check the logs."
        }
    }
}
