pipeline {
    agent any

    environment {
        ZAP_REPO_URL = "https://github.com/zaproxy/zaproxy.git"  // OWASP ZAP GitHub repository
        ZAP_HOME = "${WORKSPACE}/zap"  // Directory to clone and build ZAP
        CONTAINER_NAME = "juice-shop-${env.BUILD_ID}"  // Unique container name based on build ID
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
                    // Start OWASP Juice Shop using Docker with a unique container name
                    sh "docker run -d --name ${CONTAINER_NAME} -p 3002:3002 bkimminich/juice-shop"
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
                    ./gradlew zap-cli quick-scan http://localhost:3002
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

        stage('Stop and Remove Juice Shop Application') {
            steps {
                script {
                    // Stop and remove the dynamically named Juice Shop container
                    sh "docker stop ${CONTAINER_NAME}"
                    sh "docker rm ${CONTAINER_NAME}"
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
