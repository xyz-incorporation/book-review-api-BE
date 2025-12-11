pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Environment') {
            steps {         
                sh '''
                curl -LsSf https://astral.sh/uv/install.sh | sh
                uv venv venv
                source venv/bin/activate
                uv pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                source venv/bin/activate
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
                source venv/bin/activate
                # Check for vulnerable dependencies
                safety check --full-report || true

                # Static code analysis
                bandit -r . -f xml -o reports/bandit-report.xml || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/*.xml', allowEmptyArchive: true
                }
            }
        }

        stage('Package') {
            steps {
                sh '''
                source venv/bin/activate
                python setup.py sdist bdist_wheel
                '''
            }
        }

        stage('Archive Build Output') {
            steps {
                archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Python backend build and tests completed successfully!"
        }
        failure {
            echo "Python backend build failed!"
        }
    }
}