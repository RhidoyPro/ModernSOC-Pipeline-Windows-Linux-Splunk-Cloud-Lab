Modern SOC Home Lab – Windows 11, Ubuntu, Kali & Splunk Cloud

Author: Tashfiqul Rhidoy Prodhan
Role: Cybersecurity Student | SOC & Blue Team Enthusiast

A modern home SOC pipeline demonstrating real attack detection using:

Kali Linux (attacker)

Windows 11 (host: prodhan) with Sysmon + Splunk Universal Forwarder

Ubuntu 24.04 (host: server) with Splunk Universal Forwarder

Splunk Cloud as the SIEM

VMware Fusion on macOS for virtualization

Goal: Generate real attack telemetry (Nmap scan from Kali → Windows) and detect it in Splunk Cloud using Sysmon + Windows Event Logs.

Lab Architecture
macOS
│
├── VMware Fusion
│   ├── Kali Linux (Attacker)
│   │     - Nmap scan against Windows victim
│   │
│   ├── Windows 11 ARM (Victim)
│   │     - Sysmon64 installed
│   │     - Splunk Universal Forwarder (Windows x64)
│   │
│   └── Ubuntu 24.04 (Log Forwarder)
│         - Splunk Universal Forwarder (linux-arm64)
│
└── Splunk Cloud (SIEM)
      - Receives logs from Windows + Ubuntu


All VMs share a NAT subnet: 192.168.106.0/24

1. Ubuntu → Splunk Cloud
Install Splunk UF on Ubuntu
sudo dpkg -i splunkforwarder-10.0.1-c486717c322b-linux-arm64.deb
sudo /opt/splunkforwarder/bin/splunk start --accept-license

Add Splunk Cloud as forward-server
sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_CLOUD_FQDN>:9997 -auth admin:Prodhan1738

Enable log monitoring
sudo /opt/splunkforwarder/bin/splunk list monitor


Result: Ubuntu logs appear in Splunk Cloud → _internal index.

2. Windows 11 → Splunk Cloud
Installed Sysmon (SwiftOnSecurity config)

sysmon64.exe -accepteula -i sysmonconfig.xml

Install Splunk UF (Windows x64)

Installed using PowerShell:

Invoke-WebRequest -Uri "https://download.splunk.com/products/universalforwarder/releases/10.0.1/windows/splunkforwarder-10.0.1-c486717c322b-windows-x64.msi" -OutFile "$env:USERPROFILE\Downloads\splunkuf.msi"
Start-Process "$env:USERPROFILE\Downloads\splunkuf.msi" -Wait


Created admin user:
Username: admin
Password: Prodhan1738

Add Splunk Cloud as forward-server
splunk.exe add forward-server <SPLUNK_CLOUD_FQDN>:9997 -auth admin:Prodhan1738

Enable Windows Event Logs

C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf:

[WinEventLog://Security]
index = main

[WinEventLog://System]
index = main

[WinEventLog://Application]
index = main


Restart UF:

splunk.exe restart


Result: WinEventLog:* events appear in Splunk Cloud under index main.

3. Kali Linux Attack (Nmap → Windows)
Windows IP

Example:
192.168.106.135

Nmap scan
sudo nmap -sS 192.168.106.135


Open ports discovered:
135, 139, 445, 5357…

Result: Sysmon + Security logs generated on Windows → forwarded to Splunk Cloud → visible in WinEventLog:Security/System.

4. Splunk Cloud Detection

Search:

index=main host="prodhan" sourcetype="WinEventLog:*"


Findings:

Sysmon logs for network connections

Windows Security EventCode 5379

Windows System + Application logs

Nmap scan activity clearly visible

5. Screenshots

A folder named /screenshots/ contains:

Figure 1 – Kali Nmap scan

Figure 2 – Windows 11 logs reaching Splunk Cloud

Figure 3 – Ubuntu UF internal logs

Figure 4 – Ubuntu monitored log list

Figure 5 – Splunk Cloud telemetry (two hosts connected)

Figure 6 – Ubuntu UF service running

Summary

This home SOC lab demonstrates:

Log forwarding from Ubuntu + Windows

Sysmon deployment

Splunk Universal Forwarder configuration

Nmap attack simulation

Successful detection in Splunk Cloud

A complete end-to-end Windows → Linux → SIEM pipeline proving real SOC workflow skills.
