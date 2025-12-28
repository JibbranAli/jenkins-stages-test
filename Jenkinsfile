pipeline {
    agent any

    environment {
        IMAGE_NAME = "cicd-website"
        DEV_PORT   = "8081"
        PROD_PORT  = "8082"
        DEV_TEAM   = "dev-team@company.com"
        MANAGER    = "manager@company.com"
        OPS_TEAM   = "ops-team@company.com"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Deploy to DEV') {
            steps {
                sh '''
                docker rm -f dev-site || true
                docker run -d --name dev-site -p $DEV_PORT:80 $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Test Website') {
            steps {
                script {
                    try {
                        sh './test.sh'
                    } catch (err) {
                        emailext(
                            subject: "❌ Website Test Failed (DEV)",
                            body: "DEV deployment failed. PROD blocked.",
                            to: "$DEV_TEAM"
                        )
                        error "Stopping pipeline due to test failure"
                    }
                }
            }
        }

        stage('Manual Approval for PROD') {
            steps {
                emailext(
                    subject: "⏸ Approval Needed: Website PROD Deployment",
                    body: "Please approve PROD deployment from Jenkins UI.",
                    to: "$MANAGER"
                )
                input message: 'Approve PROD website deployment?',
                      ok: 'Approve'
            }
        }

        stage('Deploy to PROD') {
            steps {
                sh '''
                docker rm -f prod-site || true
                docker run -d --name prod-site -p $PROD_PORT:80 $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ Website Deployed Successfully to PROD",
                body: "Production website is live on port $PROD_PORT",
                to: "$OPS_TEAM"
            )
        }

        failure {
            emailext(
                subject: "❌ CI/CD Pipeline Failed",
                body: "Pipeline failed. Check Jenkins logs.",
                to: "$DEV_TEAM"
            )
        }
    }
}

