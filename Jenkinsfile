pipeline {
    agent any

    environment {
        IMAGE_NAME = "books-review-backend"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Run Tests (inside container)') {
            steps {
                sh '''
                docker run --rm \
                  -v $(pwd)/reports:/app/reports \
                  $IMAGE_NAME:$IMAGE_TAG \
                  pytest --junitxml=reports/test-results.xml
                '''
            }
            post {
                always {
                    junit 'reports/test-results.xml'
                }
            }
        }

        stage('Security / Dependency Scan') {
            steps {
                sh '''
                docker run --rm \
                  $IMAGE_NAME:$IMAGE_TAG \
                  safety check --full-report || true

                docker run --rm \
                  -v $(pwd)/reports:/app/reports \
                  $IMAGE_NAME:$IMAGE_TAG \
                  bandit -r . -f xml -o reports/bandit-report.xml || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/*.xml', allowEmptyArchive: true
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                docker rmi $IMAGE_NAME:$IMAGE_TAG || true
                '''
            }
        }
    }

    post {
        success {
            echo "Python backend build & tests completed successfully!"
        }
        failure {
            echo "Python backend build failed!"
        }
    }
}
