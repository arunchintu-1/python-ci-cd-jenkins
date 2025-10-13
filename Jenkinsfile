pipeline {
    agent any

    environment {
        IMAGE_NAME = "arunchintu-1/python-ci-cd"
    }

    stages {

        stage('Build') {
            steps {
                echo "=== Using Docker to run Python build ==="
                sh '''
                    docker run --rm -v $PWD:/app -w /app python:3.12-slim /bin/bash -c "
                        python3 -m venv venv && \
                        . venv/bin/activate && \
                        pip install --upgrade pip && \
                        if [ -f requirements.txt ]; then pip install -r requirements.txt; else echo 'No requirements.txt found'; fi
                    "
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
                withCredentials([string(credentialsId: 'dockerhub-creds', variable: 'DOCKER_TOKEN')]) {
                    sh '''
                        # Replace USERNAME with your Docker Hub username
                        echo $DOCKER_TOKEN | docker login -u arunchintu --password-stdin
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "=== Deploying Docker container ==="
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
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
