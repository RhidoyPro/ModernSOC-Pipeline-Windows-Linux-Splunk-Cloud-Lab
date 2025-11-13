# Home SOC Lab – Windows 11, Ubuntu, Kali & Splunk Cloud

**Author:** Tashfiqul Rhidoy Prodhan  

Small home lab simulating a SOC pipeline:

- **Kali Linux** attacking
- **Windows 11 (host: `prodhan`)** with Sysmon + Splunk Universal Forwarder
- **Ubuntu 24.04 server (host: `server`)** with Splunk Universal Forwarder
- **Splunk Cloud** as the SIEM

The goal: generate real attack telemetry (Nmap scan from Kali) and detect it in Splunk Cloud using Windows event logs.

---

## Lab Architecture

- VMware Fusion on macOS
  - **Kali Linux** (attacker)
  - **Windows 11 ARM** – victim workstation  
    - Sysmon64a installed with SwiftOnSecurity config  
    - Splunk UF 10.0.1 (Windows x64)
  - **Ubuntu 24.04 server** – log forwarder target  
    - Splunk UF 10.0.1 (linux-arm64)

All three VMs are on the same NAT network (192.168.106.0/24).

---

## 1. Ubuntu → Splunk Cloud

### Setup

On Ubuntu:

- Installed Splunk Universal Forwarder:

```bash
sudo dpkg -i splunkforwarder-10.0.1-c486717c322b-linux-arm64.deb
sudo /opt/splunkforwarder/bin/splunk start --accept-license
