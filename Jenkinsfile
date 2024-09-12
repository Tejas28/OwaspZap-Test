pipeline {
    agent any

    environment {
        ZAP_GITHUB_URL = "https://github.com/zaproxy/zaproxy.git"  // OWASP ZAP GitHub repository
        ZAP_HOME = "${WORKSPACE}/zap"  // Directory to clone and build OWASP ZAP
        SCAN_URL = "http://localhost:3000/#/"  // The URL you want to scan
    }

    stages {
        stage('Clone OWASP ZAP from GitHub') {
            steps {
                script {
                    // Clone the OWASP ZAP repository
                    sh """
                    git clone ${ZAP_GITHUB_URL} ${ZAP_HOME}
                    cd ${ZAP_HOME}
                    """
                }
            }
        }

        stage('Build OWASP ZAP') {
            steps {
                script {
                    // Build OWASP ZAP using Gradle
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew build
                    """
                }
            }
        }

        stage('Start OWASP ZAP in Daemon Mode') {
            steps {
                script {
                    // Start OWASP ZAP in daemon mode
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew run -Dargs="-daemon -config api.disablekey=true"
                    sleep 20  # Give ZAP time to start
                    """
                }
            }
        }

        stage('Run OWASP ZAP Scan') {
            steps {
                script {
                    // Run ZAP scan on the target URL
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew zap-cli quick-scan ${SCAN_URL}
                    """
                }
            }
        }

        stage('Generate ZAP Report') {
            steps {
                script {
                    // Generate HTML and XML reports
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew zap-cli report -o zap_report.html -f html
                    ./gradlew zap-cli report -o zap_report.xml -f xml
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

                // Archive the XML report
                archiveArtifacts artifacts: 'zap_report.xml'
            }
        }
    }

    post {
        always {
            cleanWs() // Clean workspace after build
        }
    }
}
