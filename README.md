# Modern SOC Home Lab – Windows 11, Ubuntu, Kali & Splunk Cloud

**Author:** Tashfiqul Rhidoy Prodhan  
**Role:** Cybersecurity Student · SOC / Blue Team Enthusiast

A small home SOC pipeline demonstrating **real attack telemetry** (Nmap from Kali → Windows 11) being collected and detected in **Splunk Cloud**.

## Stack

- **Kali Linux** – attacker VM (runs Nmap scan)
- **Windows 11 ARM** (host: `prodhan`) – victim workstation  
  - Sysmon64 (SwiftOnSecurity config)  
  - Splunk Universal Forwarder (Windows x64)
- **Ubuntu 24.04 LTS** (host: `server`) – log forwarder  
  - Splunk Universal Forwarder (linux-arm64)
- **Splunk Cloud** – SIEM / log analytics
- **VMware Fusion on macOS** – virtualization platform

> All VMs share the same NAT network: `192.168.106.0/24`.

---

## Lab Architecture

```text
macOS (host)
└── VMware Fusion
    ├── Kali Linux (Attacker)
    │   └── Nmap scan → Windows 11
    ├── Windows 11 ARM (Victim)
    │   ├── Sysmon64 + SwiftOnSecurity config
    │   └── Splunk Universal Forwarder (Win x64)
    └── Ubuntu 24.04 (Log Forwarder)
        └── Splunk Universal Forwarder (linux-arm64)

Splunk Cloud (SIEM)
└── Receives logs from:
    ├── Windows 11 (WinEventLog:* via UF)
    └── Ubuntu 24.04 (_internal + OS logs via UF)
1. Ubuntu → Splunk Cloud
1.1 Install Splunk Universal Forwarder (Ubuntu)
sudo dpkg -i splunkforwarder-10.0.1-c486717c322b-linux-arm64.deb

sudo /opt/splunkforwarder/bin/splunk start --accept-license

1.2 Connect Ubuntu UF to Splunk Cloud
sudo /opt/splunkforwarder/bin/splunk add forward-server \
  <SPLUNK_CLOUD_FQDN>:9997 \
  -auth admin:Prodhan1738


Replace <SPLUNK_CLOUD_FQDN> with your Splunk Cloud URL
(e.g. prd-p-21wpj.splunkcloud.com).

1.3 Verify monitored paths
sudo /opt/splunkforwarder/bin/splunk list monitor


Expected result: Ubuntu host server appears in Splunk Cloud _internal index, with UF logs and host telemetry.

2. Windows 11 → Splunk Cloud
2.1 Install Sysmon (SwiftOnSecurity config)
sysmon64.exe -accepteula -i sysmonconfig.xml

2.2 Install Splunk Universal Forwarder (Windows x64)

From elevated PowerShell:

# Download MSI
$Url     = "https://download.splunk.com/products/universalforwarder/releases/10.0.1/windows/splunkforwarder-10.0.1-c486717c322b-windows-x64.msi"
$OutFile = "$env:USERPROFILE\Downloads\splunkuf.msi"

Invoke-WebRequest -Uri $Url -OutFile $OutFile

# Launch installer
Start-Process $OutFile -Wait


During install, create the UF local admin:

Username: admin

Password: Prodhan1738 (lab-only; change in real life!)

2.3 Point UF at Splunk Cloud

In an Administrator Command Prompt, go to the Splunk UF bin folder:

cd "C:\Program Files\SplunkUniversalForwarder\bin"

splunk.exe add forward-server <SPLUNK_CLOUD_FQDN>:9997 -auth admin:Prodhan1738

2.4 Enable Windows Event Logs

Edit:

C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf


Add:

[WinEventLog://Security]
index = main

[WinEventLog://System]
index = main

[WinEventLog://Application]
index = main


Restart the UF:

splunk.exe restart


Expected result: WinEventLog:* events from host prodhan appear in the main index in Splunk Cloud.

3. Kali Linux Attack (Nmap → Windows)
3.1 Find the Windows 11 IP

On Windows (inside the VM):

ipconfig


Example output:

IPv4 Address . . . . . . . . . . : 192.168.106.135

3.2 Run Nmap from Kali

On Kali:

sudo nmap -sS 192.168.106.135


Typical open ports discovered:

135/tcp – msrpc

139/tcp – netbios-ssn

445/tcp – microsoft-ds

5357/tcp – wsdapi

Effect: This scan generates Windows Security + Sysmon network events, which the Splunk UF forwards to Splunk Cloud.

4. Splunk Cloud Detection

In Search & Reporting:

index=main host="prodhan" sourcetype="WinEventLog:*"


You should see:

Security log events (e.g. EventCode=5379)

System / Application log noise from the host

Entries that line up with the Nmap scan time window

You can tighten the search to security logs only:

index=main host="prodhan" sourcetype="WinEventLog:Security"


For Ubuntu UF internal logs:

index=_internal host="server"

5. Screenshots

The repo contains a screenshots/ folder with key figures:

Figure 1 – Kali Linux Nmap scan against Windows 11 victim

Figure 2 – Splunk Cloud search showing WinEventLog:* from host prodhan (Windows 11)

Figure 3 – Splunk _internal index events from Ubuntu forwarder host server

Figure 4 – Ubuntu Splunk UF listing monitored log files

Figure 5 – Splunk Cloud internal telemetry showing both forwarders connected

Figure 6 – Ubuntu Splunk UF service running and healthy

These screenshots visually prove the end-to-end pipeline:

Kali ➜ Windows 11 ➜ Splunk UF ➜ Splunk Cloud.

6. Summary

This lab demonstrates a modern SOC-style telemetry pipeline:

Windows + Linux hosts sending logs via Splunk Universal Forwarder

Sysmon deployment with a realistic config

Kali Nmap attack against a Windows victim

Successful detection of the activity in Splunk Cloud

It’s a compact project you can show to recruiters as proof that you:

Can build and wire up a small SIEM pipeline

Understand Windows logging (Security, System, Application, Sysmon)

Know how to simulate attacks and validate detections


If you want, next step I can help you add a tiny **“How to run this lab”** section at the top with 3–4 bullets, but this version should already look clean and professional on GitHub.
::contentReference[oaicite:1]{index=1}
