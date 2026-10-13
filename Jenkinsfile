pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out project files'
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                sh '''
                    set -eu

                    test -f app.py
                    test -f requirements.txt
                    test -f Dockerfile

                    echo "All required files exist"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu

                    docker build -t mini-devops-app:1.0 .

                    echo "Docker image built successfully"
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eu

                    # Remove any leftover test container
                    docker rm -f mini-devops-ci-test 2>/dev/null || true

                    # Start the application container
                    docker run -d \
                        --name mini-devops-ci-test \
                        mini-devops-app:1.0

                    # Always clean up the temporary container
                    trap 'docker rm -f mini-devops-ci-test >/dev/null 2>&1 || true' EXIT

                    # Retry the health check while Flask starts
                    for i in $(seq 1 20); do
                        if docker exec mini-devops-ci-test python -c \
                            "import urllib.request; r=urllib.request.urlopen('http://localhost:5000/health', timeout=2); print(r.read().decode()); assert r.status == 200" \
                            2>/dev/null; then

                            echo "Smoke test passed"
                            exit 0
                        fi

                        sleep 2
                    done

                    echo "Health check failed"
                    docker logs mini-devops-ci-test
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker build and smoke test passed!'
        }

        failure {
            echo 'FAILURE: Check the Jenkins console output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}

