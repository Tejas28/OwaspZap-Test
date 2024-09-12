pipeline {
    agent any

    environment {
        ZAP_DOCKER_IMAGE = "owasp/zap2docker-stable"  // OWASP ZAP Docker image
        SCAN_URL = "http://localhost:3000/#/"  // The URL you want to scan
    }

    stages {
        stage('Pull OWASP ZAP Docker Image') {
            steps {
                script {
                    // Pull the OWASP ZAP Docker image from Docker Hub
                    sh "docker pull ${ZAP_DOCKER_IMAGE}"
                }
            }
        }

        stage('Run OWASP ZAP Scan') {
            steps {
                script {
                    // Run the OWASP ZAP baseline scan against the given URL
                    // Replace SCAN_URL with the actual target URL you want to scan
                    sh """
                    docker run -v $WORKSPACE:/zap/wrk/:rw --network="host" ${ZAP_DOCKER_IMAGE} zap-baseline.py \
                    -t ${SCAN_URL} \
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

                // Archive the XML report for Jenkins
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
