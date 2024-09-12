pipeline {
    agent any

    environment {
        ZAP_REPO_URL = "https://github.com/zaproxy/zaproxy.git"  // OWASP ZAP GitHub repository
        ZAP_HOME = "${WORKSPACE}/zap"  // Directory to clone and build ZAP
    }

    stages {
        stage('Clone OWASP ZAP from GitHub') {
            steps {
                script {
                    // Clone the OWASP ZAP repository
                    sh """
                    git clone ${ZAP_REPO_URL} ${ZAP_HOME}
                    cd ${ZAP_HOME}
                    """
                }
            }
        }

        stage('Build OWASP ZAP') {
            steps {
                script {
                    // Build ZAP (Gradle is used for building ZAP)
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew build
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
                    // Run ZAP (it's built from source)
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew run -Dargs="-daemon -config api.disablekey=true"
                    sleep 20  # Give ZAP time to start
                    ./gradlew zap-cli status -t 120  # Wait for ZAP to be fully ready
                    ./gradlew zap-cli quick-scan http://localhost:3000
                    """
                }
            }
        }

        stage('Generate ZAP Report') {
            steps {
                script {
                    // Generate HTML and XML reports
                    sh """
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
            cleanWs() // Clean workspace after build
        }
    }
}

