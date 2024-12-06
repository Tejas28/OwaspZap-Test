#!/bin/bash
whoami

// Function to initialize parameters
initialize_parameters() {
    echo "<--Parameter Initialization-->"
    echo "The current parameters are:"
    echo "  Scan Type: $SCAN_TYPE"
    echo "  Target: $TARGET"
    echo "  Generate report: $GENERATE_REPORT"
}
 
# Function to pull and run the OWASP ZAP Docker container
setup_owasp_container() {
    echo "Pulling the latest OWASP ZAP container --> Start"
    docker pull zaproxy/zap-stable
    echo "Pulling the latest OWASP ZAP container --> End"
 
    echo "Starting OWASP ZAP container --> Start"
    docker run -dt --name owasp1 zaproxy/zap-stable /bin/bash
    echo "OWASP ZAP container started --> End"
}
 
# Function to prepare working directory inside the container
prepare_working_directory() {
    if [ "$GENERATE_REPORT" = true ]; then
        echo "Preparing working directory in OWASP ZAP container"
        docker exec owasp1 mkdir /zap/wrk
    fi
}
 
# Function to configure scan rules for OWASP Top 10
configure_scan_rules() {
    echo "Configuring OWASP ZAP to use only OWASP Top 10 scan rules"
    # Disable all scan rules
    docker exec owasp1 zap-cli --zap-url http://localhost --zap-port 8080 ascan disable-all-scanners
    # Enable only OWASP Top 10-related scan rules
    docker exec owasp1 zap-cli --zap-url http://localhost --zap-port 8080 ascan enable-scanners 40018,90018,10029,10044,10010,10016,90027,10025
    echo "Enabled OWASP Top 10 scan rules"
}
 
# Function to scan the target
perform_scan() {
    echo "Starting scan for target: $TARGET with scan type: $SCAN_TYPE"
 
    case $SCAN_TYPE in
        "Baseline")
            echo "Performing Baseline scan"
            docker exec owasp1 zap-baseline.py -t "$TARGET" -r report.html -I
            ;;
        "APIS")
            echo "Performing API scan"
            docker exec owasp1 zap-api-scan.py -t "$TARGET" -f openapi -r report.html -I
            ;;
        "Full")
            echo "Performing Full scan with OWASP Top 10 rules"
            # Start ZAP in daemon mode
            docker exec -d owasp1 zap.sh -daemon -port 8080
            # Configure scan rules for OWASP Top 10
            configure_scan_rules
            # Perform the full scan
            docker exec owasp1 zap-full-scan.py -t "$TARGET" -m 15 -r report.html -I
            ;;
        *)
            echo "Invalid scan type. Exiting..."
            exit 1
            ;;
    esac
}
 
# Function to copy the generated report to the host machine
copy_report_to_workspace() {
    echo "Copying report to the host machine"
    docker cp owasp1:/zap/wrk/report.html /home/ubuntu/Zap-Reports/report.html
}
 
# Function to clean up the container
cleanup() {
    echo "Stopping and removing the OWASP ZAP container"
    docker stop owasp1
    docker rm owasp1
    #echo "Cleaning up workspace"
    #rm -rf /home/ubuntu/Zap-Reports/*
}
 
# Main script starts here
 
# Initialize parameters
initialize_parameters
 
# Set up OWASP ZAP Docker container
setup_owasp_container
 
# Prepare working directory if needed
prepare_working_directory
 
# Perform the scan based on the chosen scan type
perform_scan
 
# Copy the report to the host workspace
copy_report_to_workspace
 
# Clean up after the scan
cleanup

