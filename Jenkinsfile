pipeline {
    agent any

    environment {
        IMAGE_NAME = "cicd-website"
        DEV_PORT   = "8081"
        PROD_PORT  = "8082"
        MAIL_TO    = "priyajoshi6721@gmail.com"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                script {
                    sh """
                        docker rm -f dev-site || true
                        docker run -d --name dev-site -p ${DEV_PORT}:80 ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Test in DEV') {
            steps {
                script {
                    try {
                        sh """
                            echo "Testing DEV environment..."
                            sleep 5
                            curl -f http://localhost:${DEV_PORT}
                        """
                    } catch (err) {
                        emailext(
                            to: "${MAIL_TO}",
                            subject: "❌ DEV TEST FAILED | Build #${BUILD_NUMBER}",
                            body: """
DEV testing failed.

Job: ${JOB_NAME}
Build: ${BUILD_NUMBER}
Stage: DEV TEST
URL: ${BUILD_URL}
"""
                        )
                        error("DEV tests failed")
                    }
                }
            }
        }

        stage('Manual Approval for PROD') {
            steps {
                script {
                    emailext(
                        to: "${MAIL_TO}",
                        subject: "⏸ Approval Required for PROD | Build #${BUILD_NUMBER}",
                        body: """
Approval required to deploy to PROD.

Job: ${JOB_NAME}
Build: ${BUILD_NUMBER}
Approve here:
${BUILD_URL}
"""
                    )

                    input(
                        message: "Approve deployment to PROD?",
                        ok: "Approve",
                        submitter: "manager",        // 🔒 ONLY MANAGER
                        submitterParameter: "APPROVED_BY"
                    )
                }
            }
        }

        stage('Deploy to PROD') {
            steps {
                script {
                    sh """
                        docker rm -f prod-site || true
                        docker run -d --name prod-site -p ${PROD_PORT}:80 ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: "${MAIL_TO}",
                subject: "✅ PROD DEPLOYMENT SUCCESS | Build #${BUILD_NUMBER}",
                body: """
Deployment successful.

Job: ${JOB_NAME}
Build: ${BUILD_NUMBER}
Environment: PROD
Approved by: ${APPROVED_BY}

URL: ${BUILD_URL}
"""
            )
        }

        failure {
            emailext(
                to: "${MAIL_TO}",
                subject: "❌ PIPELINE FAILED | Build #${BUILD_NUMBER}",
                body: """
Pipeline failed.

Job: ${JOB_NAME}
Build: ${BUILD_NUMBER}

Check console logs:
${BUILD_URL}
"""
            )
        }

        always {
            echo "Pipeline execution finished."
        }
    }
}
