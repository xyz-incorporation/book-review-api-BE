pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Build: successful'
            }
        }

        stage('Test') {
            steps {
                echo 'Test: successful'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying to STAGING...'
                    sh '''
                      echo "Staging deployment successful"
                    '''

                    echo 'Deploying to PRODUCTION...'
                    sh '''
                      echo "Production deployment successful"
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully' }
        failure { echo 'Pipeline failed' }
    }
}
