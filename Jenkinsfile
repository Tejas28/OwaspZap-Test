pipeline {
    agent any

    environment {
        ZAP_HOME = "/path/to/owasp/zap" // Adjust this if OWASP ZAP is installed natively
    }

    stages {
        stage('Checkout') {
            steps {
                // No need to checkout the code since we're pulling the Juice Shop image from Docker
                echo 'No SCM checkout required for Juice Shop'
            }
        }

        stage('Start Juice Shop Application') {
            steps {
                script {
                    // Start OWASP Juice Shop using Docker
                    sh 'docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop'
                    // Wait for the app to be fully started
                    sleep 30
                }
            }
        }

        stage('Run OWASP ZAP Scan') {
            steps {
                script {
                    // Run OWASP ZAP against Juice Shop
                    // Use OWASP ZAP Docker image to scan the application running at http://localhost:3000
                    sh """
                    docker run -v $WORKSPACE:/zap/wrk/:rw --network="host" -t owasp/zap2docker-stable zap-baseline.py \
                        -t http://localhost:3000 \
                        -r zap_report.html -w zap_report.xml
                    """
                }
            }
        }

        stage('Publish ZAP Report') {
            steps {
                // Publish the ZAP HTML report in Jenkins
                publishHTML (target: [
                    allowMissing: false,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'zap_report.html',
                    reportName: 'OWASP ZAP Report'
                ])

                // Archive the XML report for Jenkins to analyze
                archiveArtifacts artifacts: 'zap_report.xml'
            }
        }

        stage('Stop Juice Shop Application') {
            steps {
                script {
                    // Stop and remove the Juice Shop Docker container
                    sh 'docker stop juice-shop'
                    sh 'docker rm juice-shop'
                }
            }
        }
    }

    post {
        always {
            // Clean the workspace after the build
            cleanWs()
        }
    }
}
