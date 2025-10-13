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
                    # Use Docker to create a virtual environment and install dependencies
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
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
