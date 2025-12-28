pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to DEV') {
            steps {
                sh 'chmod +x app.sh'
                sh './app.sh'
            }
        }

        stage('Testing') {
            steps {
                sh 'echo Tests Passed'
            }
        }

        stage('PROD Approval') {
            steps {
                input message: 'Approve PROD deployment',
                      ok: 'Approve'
            }
        }

        stage('Deploy to PROD') {
            steps {
                sh 'echo Deployed to PROD'
            }
        }
    }
}
