# port-scan.sh


#!/bin/bash

TARGET=$1

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target-domain-or-ip>"
    exit 1
fi

echo "=== NMAP ==="
nmap -Pn -T4 $TARGET

echo ""
echo "=== MASSCAN ==="
sudo masscan $TARGET -p1-1000 --rate=1000

echo ""
echo "=== RUSTSCAN ==="
rustscan -a $TARGET --ulimit 5000

echo ""
echo "=== UNICORNSCAN ==="
unicornscan -Iv $TARGET:a

echo ""
echo "=== HPING3 (SYN scan) ==="
hping3 -S $TARGET -p 80 -c 1

echo ""
echo "=== NETCAT (common ports) ==="
for port in 21 22 23 25 53 80 110 139 143 443 445 3389; do
    (echo > /dev/tcp/$TARGET/$port) >/dev/null 2>&1 && echo "Port $port is open"
done

echo ""
echo "=== ZMAP (fast scan, root needed) ==="
sudo zmap -p 80 $TARGET -o - 2>/dev/null

echo ""
echo "=== SCANLESS (online scanners) ==="
scanless -t $TARGET

echo ""
echo "=== FSCAN ==="
fscan -h $TARGET

echo ""
echo "=== PNSCAN (common ports) ==="
pnscan -r 21-443 $TARGET

echo ""
echo "=== TCPING (ping to port 80) ==="
tcping -t 1 $TARGET 80

echo ""
echo "=== ARP-SCAN (LAN only) ==="
sudo arp-scan $TARGET

echo ""
echo "=== NBTSCAN (NetBIOS) ==="
nbtscan $TARGET

echo ""
echo "=== TELNET (manual check port 80) ==="
(echo quit | telnet $TARGET 80 2>/dev/null | grep Connected) && echo "Port 80 is open"

echo ""
echo "=== SOCKET TEST (bash loop, 1-100) ==="
for port in {1..100}; do
    timeout 1 bash -c "echo > /dev/tcp/$TARGET/$port" 2>/dev/null && echo "Port $port is open"
done

echo ""
echo "=== ENUM4LINUX (SMB ports) ==="
enum4linux -p $TARGET

echo ""
echo "=== OPENVAS / NESSUS / METASPLOIT ==="
echo "These are advanced scanners, please run them manually for full scan."

echo ""
echo "Port scan complete!"
