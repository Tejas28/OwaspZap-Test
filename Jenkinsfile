pipeline {
    agent any
    environment {
        SCAN_TYPE = "Full" // Set this to "Baseline", "APIS", or "Full" as needed
        TARGET = "https://example.com" // Replace with your target URL
        GENERATE_REPORT = true // Set to true if a report needs to be generated
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
                    sh 'docker run -dt --name owasp1 zaproxy/zap-stable /bin/bash'
                    echo "OWASP ZAP container started --> End"
                }
            }
        }

        stage('Prepare Working Directory') {
            when {
                expression { env.GENERATE_REPORT == 'true' }
            }
            steps {
                script {
                    echo "Preparing working directory in OWASP ZAP container"
                    sh 'docker exec owasp1 mkdir -p /zap/wrk'
                }
            }
        }

        stage('Configure Scan Rules for OWASP Top 10') {
            when {
                expression { env.SCAN_TYPE == 'Full' }
            }
            steps {
                script {
                    echo "Configuring OWASP ZAP to use only OWASP Top 10 scan rules"
                    sh 'docker exec owasp1 zap-cli --zap-url http://localhost --zap-port 8080 ascan disable-all-scanners'
                    sh 'docker exec owasp1 zap-cli --zap-url http://localhost --zap-port 8080 ascan enable-scanners 40018,90018,10029,10044,10010,10016,90027,10025'
                    echo "Enabled OWASP Top 10 scan rules"
                }
            }
        }

        stage('Perform Scan') {
            steps {
                script {
                    echo "Starting scan for target: ${env.TARGET} with scan type: ${env.SCAN_TYPE}"

                    if (env.SCAN_TYPE == 'Baseline') {
                        echo "Performing Baseline scan"
                        sh "docker exec owasp1 zap-baseline.py -t ${env.TARGET} -r report.html -I"
                    } else if (env.SCAN_TYPE == 'APIS') {
                        echo "Performing API scan"
                        sh "docker exec owasp1 zap-api-scan.py -t ${env.TARGET} -f openapi -r report.html -I"
                    } else if (env.SCAN_TYPE == 'Full') {
                        echo "Performing Full scan with OWASP Top 10 rules"
                        sh 'docker exec -d owasp1 zap.sh -daemon -port 8080'
                        sh "docker exec owasp1 zap-full-scan.py -t ${env.TARGET} -m 15 -r report.html -I"
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
                    echo "Copying report to the host machine"
                    sh 'docker cp owasp1:/zap/wrk/report.html /home/ubuntu/Zap-Reports/report.html'
                }
            }
        }
    }
    post {
        always {
                steps {
                    script {
                        echo "Stopping and removing the OWASP ZAP container"
                        sh 'docker stop owasp1'
                        sh 'docker rm owasp1'
                    }
                }
            
        }
    }
}
