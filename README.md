
UNIVERSITY OF MANAGEMENT & TECHNOLOGY
Department of Computer Science


Setup & Lab Guide
Firewall & Intrusion Detection System
Step-by-Step Installation & Configuration Guide
 Lab Overview
This guide provides step-by-step instructions for setting up a complete Firewall and IDS lab environment on Windows 10. Follow each section in order for best results.

Step	Task	Estimated Time
1	Install Nmap on Windows 10	5 minutes
2	Configure Windows Defender Firewall Rules	10 minutes
3	Install Snort IDS	15 minutes
4	Configure Snort — Custom Rules	10 minutes
5	Connect Kali Linux VM to same network	10 minutes
6	Run Nmap Scan — Before & After	5 minutes
7	Run Snort IDS & Generate Alerts	10 minutes
8	Capture Traffic in Wireshark	5 minutes
9	Run SOC Dashboard	5 minutes

Step 1: Install Nmap on Windows 10
1.1 Download
1.	Open browser — go to https://nmap.org/download.html
2.	Find "Microsoft Windows binaries" section
3.	Download "Latest stable release self-installer" (.exe file)
4.	Current version: nmap-7.99-setup.exe

1.2 Install
5.	Run the downloaded .exe file as Administrator
6.	Click Next > I Agree > Next > Next > Install
7.	When Npcap installer appears — click Install (keep defaults)
8.	Click Finish when complete

1.3 Verify Installation
  Open CMD as Administrator and run:

nmap --version

Expected Output:
Nmap version 7.99 ( https://nmap.org )
Platform: i686-pc-windows-windows

Step 2: Configure Windows Defender Firewall
2.1 Open CMD as Administrator
9.	Press Windows key + X
10.	Click "Command Prompt (Admin)" or "Windows PowerShell (Admin)"
11.	Click Yes on UAC prompt

2.2 Create Firewall Rules
Run each command one by one:

REM Block Telnet (Port 23)
netsh advfirewall firewall add rule name="Block Telnet" protocol=TCP dir=in localport=23 action=block

REM Allow HTTP (Port 80)
netsh advfirewall firewall add rule name="Allow HTTP" protocol=TCP dir=in localport=80 action=allow

REM Allow HTTPS (Port 443)
netsh advfirewall firewall add rule name="Allow HTTPS" protocol=TCP dir=in localport=443 action=allow

REM Block FTP (Port 21)
netsh advfirewall firewall add rule name="Block FTP" protocol=TCP dir=in localport=21 action=block

REM Block MySQL (Port 3306)
netsh advfirewall firewall add rule name="Block MySQL" protocol=TCP dir=in localport=3306 action=block

REM Enable Dropped Packet Logging
netsh advfirewall set allprofiles logging droppedconnections enable

2.3 Allow ICMP (for Ping Testing)
netsh advfirewall firewall add rule name="Allow ICMP" protocol=icmpv4:8,any dir=in action=allow

2.4 Verify Rules
netsh advfirewall firewall show rule name="Block Telnet"
netsh advfirewall firewall show rule name="Allow HTTP"
netsh advfirewall firewall show rule name="Allow HTTPS"

  Expected: Each rule shows Enabled: Yes, Action: Block or Allow as configured

Step 3: Install Snort IDS
3.1 Download
12.	Go to https://www.snort.org/downloads
13.	Download Snort 2.9.x Windows installer (.exe)
14.	Also download WinPcap from https://www.winpcap.org/install

3.2 Install WinPcap First
15.	Run WinPcap_4_1_3.exe
16.	Next > Next > Install > Finish

3.3 Install Snort
17.	Run Snort installer as Administrator
18.	Next > I Agree > Next > Next > Install
19.	Default install path: C:\Snort
20.	Click Finish

3.4 Verify Snort Installation
cd C:\Snort\bin
snort --version

Expected Output:
,,_     -*> Snort! <*-
o"  )~   Version 2.9.20-WIN64 GRE (Build 82)

3.5 Check Available Interfaces
snort -W

Expected: List of network interfaces with IP addresses
Find the interface with your Wi-Fi IP (e.g., 192.168.100.6)
Note the Index number (e.g., 5)

Step 4: Configure Snort
4.1 Edit snort.conf
Open Notepad and edit: C:\Snort\etc\snort.conf
Make these changes:

REM Find and replace these lines:

ipvar HOME_NET 192.168.100.0/24
ipvar EXTERNAL_NET !$HOME_NET

dynamicpreprocessor directory C:\Snort\lib\snort_dynamicpreprocessor
dynamicengine C:\Snort\lib\snort_dynamicengine\sf_engine.dll

REM Comment out this line (add # at start):
# dynamicdetection directory /usr/local/lib/snort_dynamicrules

4.2 Create Required Files
REM Create empty rules files
echo. > C:\Snort\rules\local.rules
echo. > C:\Snort\rules\custom.rules

4.3 Add Custom Rules
Open notepad C:\Snort\rules\custom.rules and paste these 6 rules:

alert icmp any any -> any any (msg:"ICMP Ping Detected"; itype:8; sid:1001; rev:1;)
alert tcp any any -> any 80 (msg:"Suspicious HTTP Keyword - cmd"; content:"cmd"; sid:1002; rev:1;)
alert tcp any any -> any any (msg:"Nmap Port Scan Detected"; flags:S; sid:1003; rev:1;)
alert tcp any any -> any 23 (msg:"Telnet Access Attempt"; flags:S; sid:1004; rev:1;)
alert tcp any any -> any any (msg:"SYN Flood DoS Attempt"; flags:S; sid:1005; rev:1;)
alert tcp any any -> any 21 (msg:"FTP Brute Force Attempt"; content:"USER"; sid:1006; rev:1;)

  Save the file with Ctrl+S — make sure it is saved as custom.rules not custom.rules.txt

4.4 Update snort.conf to Include Custom Rules
In snort.conf, find the line: include $RULE_PATH/local.rules
Add below it: include $RULE_PATH/custom.rules

Step 5: Network Setup — Kali Linux on Same Network
5.1 Connect Both Machines to Same Network
Both Windows 10 and Kali Linux must be on the same network for attack simulation.

Option	How	Best For
Home Wi-Fi	Connect both to same home router	Recommended
Mobile Hotspot	Create hotspot on phone, connect both	No home router
VirtualBox Bridged	Set Kali adapter to Bridged — same Wi-Fi	VM users

5.2 Verify Connectivity
REM On Windows — check IP:
ipconfig
(Note down IPv4 Address — e.g., 192.168.100.6)

REM On Kali — check IP:
ip addr show
(Note down inet address — e.g., 192.168.100.15)

REM Test ping from Kali to Windows:
ping 192.168.100.6
(Should get replies — 64 bytes from 192.168.100.6)
  If ping fails: Check both are on same network. If on university Wi-Fi, switch to home Wi-Fi or mobile hotspot — university Wi-Fi blocks device-to-device communication.

Step 6: Run Nmap Scans
6.1 Before Firewall Rules
Run this from Kali Linux BEFORE applying firewall rules:
nmap -sS -sV -p 21,22,23,80,443,3306 192.168.100.6

Expected: Most ports shown as OPEN

6.2 After Firewall Rules
Run same command AFTER applying Windows Defender Firewall rules:
nmap -sS --min-rate 200 192.168.100.6

Expected: All targeted ports shown as FILTERED

  Screenshot both results for documentation. The difference proves firewall is working correctly.

Step 7: Run Snort IDS & Generate Alerts
7.1 Start Snort
On Windows CMD (Run as Administrator):
cd C:\Snort\bin
snort -i 5 -c C:\Snort\etc\snort.conf -l C:\Snort\log -A fast

Note: Replace -i 5 with your actual interface number from snort -W
Wait for: Commencing packet processing (pid=XXXX)

7.2 Minimize CMD — Do NOT Close
  Keep Snort running in background. Open a new CMD window for other commands.

7.3 Simulate Attacks from Kali
REM ICMP Ping Sweep:
ping -c 100 192.168.100.6

REM Nmap Port Scan:
nmap -sS --min-rate 200 192.168.100.6

REM Telnet Attempt:
telnet 192.168.100.6

7.4 Check Snort Alerts
type C:\Snort\log\alert.ids

Expected Output Example:
06/11-23:30:12.648105  [**] [1:1003:1] Nmap Port Scan Detected [**]
[Priority: 0] {TCP} 192.168.100.15 -> 192.168.100.6:445

Step 8: Capture Traffic in Wireshark
8.1 Start Capture
21.	Open Wireshark
22.	Select Wi-Fi interface (the one with 192.168.100.6)
23.	Double-click to start capture

8.2 Apply Display Filters
While Kali sends attacks, apply these filters in Wireshark:
Filter	What It Shows
icmp	All ICMP ping packets
icmp.type==8	Only ICMP Echo Requests (ping)
tcp.flags.syn==1 && tcp.flags.ack==0	SYN packets — port scan / SYN flood
tcp.port==23	Telnet traffic
tcp.port==80 && frame contains "cmd"	HTTP with cmd keyword
tcp.port==21	FTP traffic

8.3 Save Capture
24.	Click File > Save As
25.	Save as: wireshark_capture.pcapng
26.	Take screenshots of each filter for documentation

Step 9: Run SOC Dashboard
9.1 Prerequisites
REM Install Python dependencies:
pip install streamlit pandas plotly

9.2 Setup Dashboard Files
Create folder C:\SOC_Dashboard and place these files inside:
•	app.py — Main dashboard application
•	database.py — SQLite database setup
•	auth.py — Authentication module
•	requirements.txt — Python dependencies

9.3 Run Dashboard
cd C:\SOC_Dashboard
python -m streamlit run app.py

Dashboard will open at: http://localhost:8501

9.4 Login Credentials
Username	Password	Role	Access
admin	admin123	Admin	Full access — all pages
analyst	analyst123	Analyst	Security monitoring pages
viewer	viewer123	Viewer	Read-only access

9.5 Live Snort Feed Setup
For real-time Snort alerts in dashboard:
27.	Make sure Snort is running (Step 7)
28.	Open dashboard > Navigate to Live Snort Feed
29.	Click Refresh Alerts button
30.	Dashboard reads C:\Snort\log\alert.ids automatically

Troubleshooting Guide
Problem	Cause	Solution
Snort not recognized	PATH not set	Use: cd C:\Snort\bin then snort command
alert.ids is empty	No matching traffic	Disable firewall temporarily, run attacks again
Kali cannot ping Windows	Wrong network	Use home Wi-Fi or mobile hotspot instead of university Wi-Fi
Snort config error	Missing rules files	Run: echo. > C:\Snort\rules\local.rules
Dashboard not opening	Port in use	Try: python -m streamlit run app.py --server.port 8502
Nmap not found	PATH issue	Run from C:\Program Files (x86)\Nmap\nmap.exe

Final Verification Checklist
#	Task	Command to Verify	Status
1	Nmap installed	nmap --version	
2	Snort installed	cd C:\Snort\bin && snort --version	
3	Block Telnet rule active	netsh advfirewall firewall show rule name="Block Telnet"	
4	Allow HTTP rule active	netsh advfirewall firewall show rule name="Allow HTTP"	
5	Custom rules file created	type C:\Snort\rules\custom.rules	
6	Snort starts without errors	snort -i 5 -c C:\Snort\etc\snort.conf -l C:\Snort\log -A fast	
7	Kali can ping Windows	ping 192.168.100.6 (from Kali)	
8	Nmap scan shows filtered ports	nmap -sS 192.168.100.6 (from Kali)	
9	Snort generates alerts	type C:\Snort\log\alert.ids	
10	Wireshark captures packets	icmp filter shows packets	
11	Dashboard runs	http://localhost:8501	
12	Live Snort Feed works	Dashboard > Live Snort Feed > Refresh	


Setup Guide prepared by: Malik Abubakar | F2024408134 | 4th Semester BS Cybersecurity | UMT Lahore | June 2026
