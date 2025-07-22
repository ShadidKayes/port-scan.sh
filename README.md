#port-vurl.sh

#!/bin/bash

TARGET=$1
REPORT_DIR="port_vuln_reports"
MAIN_REPORT="$REPORT_DIR/combined_port_report.txt"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target-domain-or-ip>"
    exit 1
fi

mkdir -p $REPORT_DIR
echo "Port Vulnerability Scan Report for $TARGET" > $MAIN_REPORT
echo "Scan Date: $(date)" >> $MAIN_REPORT
echo "----------------------------------------" >> $MAIN_REPORT

echo "=== NMAP Vulnerability Scan ==="
nmap -Pn -p- --script vuln $TARGET -oN $REPORT_DIR/nmap_vuln.txt
echo -e "\n--- NMAP Vulnerability Scan ---\n" >> $MAIN_REPORT
cat $REPORT_DIR/nmap_vuln.txt >> $MAIN_REPORT

echo "=== Nuclei (port templates) ==="
nuclei -u $TARGET -tags network,port -o $REPORT_DIR/nuclei_port_vuln.txt
echo -e "\n--- Nuclei (port templates) ---\n" >> $MAIN_REPORT
cat $REPORT_DIR/nuclei_port_vuln.txt >> $MAIN_REPORT

echo "=== Vulners ==="
vulners -t $TARGET > $REPORT_DIR/vulners.txt
echo -e "\n--- Vulners ---\n" >> $MAIN_REPORT
cat $REPORT_DIR/vulners.txt >> $MAIN_REPORT

echo "=== Metasploit (manual, SMB example) ==="
echo "use auxiliary/scanner/smb/smb_version; set RHOSTS $TARGET; run" | msfconsole -q > $REPORT_DIR/msf_smb.txt
echo -e "\n--- Metasploit SMB Version ---\n" >> $MAIN_REPORT
cat $REPORT_DIR/msf_smb.txt >> $MAIN_REPORT

echo "=== SSLScan (if applicable) ==="
sslscan $TARGET > $REPORT_DIR/sslscan.txt
echo -e "\n--- SSLScan ---\n" >> $MAIN_REPORT
cat $REPORT_DIR/sslscan.txt >> $MAIN_REPORT

echo ""
echo "All port vulnerability scans complete!"
echo "Combined port report saved at: $MAIN_REPORT"
