pipeline {
    agent any
    environment {
        SCAN_TYPE = "Full" // Set this to "Baseline", "APIS", or "Full" as needed
        TARGET = "https://dev-mdashboard.dev.gokwik.in/login#" // Replace with your target URL
        GENERATE_REPORT = true // Set to true if a report needs to be generated
        ZAP_API = "http://localhost:8080" // ZAP API endpoint
    }
    stages {
        stage('Initialize Parameters') {
            steps {
                script {
                    echo "<--Parameter Initialization-->"
                    echo "The current parameters are:"
                    echo "  Scan Type: ${env.SCAN_TYPE}"
                    echo "  Target: ${env.TARGET}"
                    echo "  Generate report: ${env.GENERATE_REPORT}"
                }
            }
        }

        stage('Setup OWASP ZAP Container') {
            steps {
                script {
                    echo "Pulling the latest OWASP ZAP container --> Start"
                    sh 'docker pull zaproxy/zap-stable'
                    echo "Pulling the latest OWASP ZAP container --> End"

                    echo "Starting OWASP ZAP container --> Start"
                    sh 'docker run -dt --name owasp2 -p 8080:8080 zaproxy/zap-stable zap.sh -daemon -port 8080'
                    echo "OWASP ZAP container started --> End"
                }
            }
        }

        stage('Perform Scan') {
            steps {
                script {
                    echo "Starting scan for target: ${env.TARGET} with scan type: ${env.SCAN_TYPE}"

                    if (env.SCAN_TYPE == 'Baseline') {
                        echo "Performing Baseline scan"
                        sh "docker exec owasp2 zap-baseline.py -t \"${env.TARGET}\" -r report.html -I"
                    } else if (env.SCAN_TYPE == 'APIS') {
                        echo "Performing API scan"
                        sh "docker exec owasp2 zap-api-scan.py -t ${env.TARGET} -f openapi -r report.html -I"
                    } else if (env.SCAN_TYPE == 'Full') {
                        echo "Configuring OWASP ZAP to use only OWASP Top 10 scan rules for Full scan"
                        sh "curl \"${env.ZAP_API}/JSON/ascan/action/disableAllScanners/\""
                        sh "curl \"${env.ZAP_API}/JSON/ascan/action/enableScanners/?ids=40018,90018,10029,10044,10010,10016,90027,10025\""
                        echo "Enabled OWASP Top 10 scan rules"

                        echo "Performing Full scan with OWASP Top 10 rules"
                        sh "curl \"${env.ZAP_API}/JSON/spider/action/scan/?url=${env.TARGET}&maxChildren=10\""
                        sh "curl \"${env.ZAP_API}/JSON/ascan/action/scan/?url=${env.TARGET}\""
                    } else {
                        error "Invalid scan type: ${env.SCAN_TYPE}. Exiting..."
                    }
                }
            }
        }

        stage('Copy Report to Workspace') {
            when {
                expression { env.GENERATE_REPORT == 'true' }
            }
            steps {
                script {
                    echo "Retrieving the report via REST API"
                    sh "curl \"${env.ZAP_API}/OTHER/core/other/htmlreport/\" -o /home/ubuntu/Zap-Reports/report.html"
                }
            }
        }
    }
    post {
        always {
            steps {
                script {
                    echo "Stopping and removing the OWASP ZAP container"
                    sh 'docker stop owasp2'
                    sh 'docker rm owasp2'
                }
            }
        }
    }
}
