pipeline {
    agent any

    environment {
        IMAGE_NAME = "books-review-backend"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_TOKEN = credentials('sonarqube-token-be')
        DOCKER_REGISTRY = "kumarsai13/book-review-be"
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
                  pytest \
                    --junitxml=reports/test-results.xml \
                    --cov=. \
                    --cov-report=xml:reports/coverage.xml
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

        //stage('SonarQube Analysis') {
            //steps {
                //withSonarQubeEnv('SonarQubeServer') {
                    //sh '''
                    //sonar-scanner \
                      //-Dsonar.projectKey=books-review-backend \
                      //-Dsonar.sources=. \
                      //-Dsonar.host.url=${SONAR_HOST_URL} \
                      //-Dsonar.token=${SONAR_TOKEN} \
                      //-Dsonar.python.coverage.reportPaths=reports/coverage.xml \
                      //-Dsonar.python.bandit.reportPaths=reports/bandit-report.xml
                    //'''
                //}
            //}
        //}

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image \
                  --scanners vuln \
                  --severity CRITICAL \
                  --exit-code 1 \
                  --no-progress \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
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
