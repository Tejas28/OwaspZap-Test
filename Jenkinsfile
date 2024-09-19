def scan_type
def target
 pipeline {
    agent any
  
    parameters {
        choice  choices: ['Baseline', 'APIS', 'Full'],
                 description: 'Type of scan that is going to perform inside the container',
                 name: 'SCAN_TYPE'

        string defaultValue: 'http://localhost:3000/',
                 description: 'Target URL to scan',
                 name: 'TARGET'

        booleanParam defaultValue: true,
                 description: 'Parameter to know if wanna generate report.',
                 name: 'GENERATE_REPORT'
    }
    stages {
        stage('Pipeline Info') {
            steps {
                script {
                    echo '<--Parameter Initialization-->'
                    echo """
                         The current parameters are:
                             Scan Type: ${params.SCAN_TYPE}
                             Target: ${params.TARGET}
                             Generate report: ${params.GENERATE_REPORT}
                         """
                }
            }
        }

        stage('Setting up OWASP ZAP docker container') {
            steps {
                echo 'Pulling up last OWASP ZAP container --> Start'
                sh 'docker pull zaproxy/zap-stable'
                echo 'Pulling up last VMS container --> End'
                echo 'Starting container --> Start'
                sh 'docker run -dt --name owasp1 zaproxy/zap-stable /bin/bash '
            }
        }

        stage('Prepare wrk directory') {
            when {
                environment name : 'GENERATE_REPORT', value: 'true'
            }
            steps {
                script {
                    sh '''
                             docker exec owasp1 \
                             mkdir /zap/wrk
                         '''
                }
            }
        }

        stage('Scanning target on owasp container') {
            steps {
                script {
                    scan_type = "${params.SCAN_TYPE}"
                    echo "----> scan_type: $scan_type"
                    target = "${params.TARGET}"
                    if (scan_type == 'Baseline') {
                        sh """
                             docker exec owasp1 \
                             zap-baseline.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                    }
                     else if (scan_type == 'APIS') {
                        sh """
                             docker exec owasp1 \
                             zap-api-scan.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                     }
                     else if (scan_type == 'Full') {
                        sh """
                             docker exec owasp1 \
                             zap-full-scan.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                     }
                     else {
                        echo 'Something went wrong...'
                     }
                }
            }
        }
        stage('Copy Report to Workspace') {
            steps {
                script {
                    sh '''
                         docker cp owasp1:/zap/wrk/report.html /home/tejas/Zap-Reports/report.html
                     '''
                }
            }
        }

    }
    post {
        always {
            echo 'Removing container'
            sh '''
                     docker stop owasp1
                     docker rm owasp1
                 '''
            cleanWs()
        }
    }
 } 
