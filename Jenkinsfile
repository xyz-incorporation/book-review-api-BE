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
                sh '''#!/bin/bash
                curl -LsSf https://astral.sh/uv/install.sh | sh
                PATH="/var/lib/jenkins/.local/bin:$PATH"
                export PATH
                export PATH
                uv venv
                uv pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''#!/bin/bash
                uv run pytest --junitxml=reports/test-results.xml
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
                sh '''#!/bin/bash
                # Check for vulnerable dependencies
                uv run safety check --full-report || true

                # Static code analysis
                uv run bandit -r . -f xml -o reports/bandit-report.xml || true
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
                sh '''#!/bin/bash
                uv run python setup.py sdist bdist_wheel
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