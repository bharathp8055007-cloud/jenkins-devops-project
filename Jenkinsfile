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
                    test -f app.py
                    test -f requirements.txt
                    test -f Dockerfile
                    echo "All required files exist"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t mini-devops-app:1.0 .'
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eu
                    docker rm -f mini-devops-ci-test 2>/dev/null || true
                    trap 'docker rm -f mini-devops-ci-test >/dev/null 2>&1 || true' EXIT

                    docker run -d --name mini-devops-ci-test \
                        -p 127.0.0.1:5001:5000 mini-devops-app:1.0

                    for i in $(seq 1 20); do
                        if curl -fsS http://localhost:5001/health; then
                            echo "Health check passed"
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
    }
}

