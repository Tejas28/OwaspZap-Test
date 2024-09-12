pipeline {
    agent any

    environment {
        ZAP_VERSION = "2.13.0" // Specify the ZAP version you want to use
        ZAP_DOWNLOAD_URL = "https://github.com/zaproxy/zaproxy/releases/download/v${ZAP_VERSION}/ZAP_${ZAP_VERSION}_Linux.tar.gz"
        ZAP_HOME = "${WORKSPACE}/zap" // ZAP installation path
    }

    stages {
        stage('Download OWASP ZAP') {
            steps {
                script {
                    // Download and extract ZAP from GitHub
                    sh """
                    curl -L -o zap.tar.gz ${ZAP_DOWNLOAD_URL}
                    mkdir -p ${ZAP_HOME}
                    tar -xzf zap.tar.gz -C ${ZAP_HOME} --strip-components=1
                    """
                }
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
                    // Run the ZAP baseline scan against the Juice Shop
                    sh """
                    ${ZAP_HOME}/zap.sh -daemon -config api.disablekey=true
                    sleep 20  # Give ZAP time to start
                    ${ZAP_HOME}/zap-cli status -t 120  # Wait for ZAP to be fully ready
                    ${ZAP_HOME}/zap-cli quick-scan http://localhost:3000
                    """
                }
            }
        }

        stage('Generate ZAP Report') {
            steps {
                script {
                    // Generate HTML and XML reports using ZAP
                    sh """
                    ${ZAP_HOME}/zap-cli report -o zap_report.html -f html
                    ${ZAP_HOME}/zap-cli report -o zap_report.xml -f xml
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
            cleanWs() // Clean the workspace after the build
        }
    }
}
