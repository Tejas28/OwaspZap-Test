pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-1.11.0-openjdk-amd64
"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        ZAP_TAG = "v2.11.1"
        ZAP_REPO_URL = "https://github.com/zaproxy/zaproxy.git"
        ZAP_HOME = "${WORKSPACE}/zap"
    }

    stages {
        stage('Check Java Version') {
            steps {
                script {
                    sh 'java -version'
                }
            }
        }

        stage('Clone OWASP ZAP from GitHub') {
            steps {
                script {
                    // Clone the OWASP ZAP repository and checkout the specified stable release tag
                    sh """
                    git clone ${ZAP_REPO_URL} ${ZAP_HOME}
                    cd ${ZAP_HOME}
                    git checkout ${ZAP_TAG}
                    """
                }
            }
        }

        stage('Build OWASP ZAP') {
            steps {
                script {
                    // Build OWASP ZAP using Gradle and add verbose logging
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew build --info --stacktrace
                    """
                }
            }
        }

        stage('Stop and Remove Existing Juice Shop Container') {
            steps {
                script {
                    // Stop and remove any existing "juice-shop" container
                    sh """
                    if [ \$(docker ps -aq -f name=juice-shop) ]; then
                        docker stop juice-shop
                        docker rm juice-shop
                    fi
                    """
                }
            }
        }

        stage('Start Juice Shop Application') {
            steps {
                script {
                    // Start OWASP Juice Shop using Docker
                    sh 'docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop'
                    sleep 30
                }
            }
        }

        stage('Run OWASP ZAP Scan') {
            steps {
                script {
                    sh """
                    cd ${ZAP_HOME}
                    ./gradlew run -Dargs="-daemon -config api.disablekey=true"
                    sleep 20
                    ./gradlew zap-cli status -t 120
                    ./gradlew zap-cli quick-scan http://localhost:3000
                    """
                }
            }
        }

        stage('Generate ZAP Report') {
            steps {
                script {
                    sh """
                    ./gradlew zap-cli report -o zap_report.html -f html
                    ./gradlew zap-cli report -o zap_report.xml -f xml
                    """
                }
            }
        }

        stage('Publish ZAP Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'zap_report.html',
                    reportName: 'OWASP ZAP Report'
                ])

                archiveArtifacts artifacts: 'zap_report.xml'
            }
        }

        stage('Stop and Remove Juice Shop Application') {
            steps {
                script {
                    sh 'docker stop juice-shop'
                    sh 'docker rm juice-shop'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

