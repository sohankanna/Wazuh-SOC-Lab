# Wazuh-SOC-Lab
A fully functional Security Operations Center (SOC) home lab using Wazuh for SIEM/XDR, featuring automated detection and response capabilities.


# Wazuh SIEM/XDR Home Lab Project

<p align="center">
  <img src="YOUR_ARCHITECTURE_DIAGRAM_URL_HERE" width="700">
</p>

## Overview

This project demonstrates the deployment and configuration of a comprehensive Security Operations Center (SOC) using the open-source Wazuh SIEM/XDR platform. The goal was to build a multi-machine virtual lab to simulate real-world security scenarios, including proactive vulnerability scanning, threat detection, and automated incident response.

---

## Lab Architecture

The lab was built using Oracle VirtualBox and consists of four virtual machines on a dual-homed network (NAT Network for inter-VM communication and a Host-Only network for management):

- **Wazuh Server:** Ubuntu Server running the central Wazuh manager, indexer, and dashboard.
- **Windows Endpoint:** A Windows 11 Pro machine acting as a corporate workstation.
- **Linux Endpoint:** An Ubuntu Server running an Apache web server, acting as a public-facing asset.
- **Attacker Machine:** A Kali Linux instance used to simulate threats.

_**[Screenshot 1: Your VirtualBox Manager window showing the 4 VMs]**_

---

## Core Capabilities Demonstrated

### 1. Advanced Endpoint Monitoring with Sysmon

I enhanced endpoint visibility on the Windows agent by integrating Sysmon. This allowed for the detection of suspicious process creation, such as the use of legitimate tools for malicious purposes ("Living off the Land"). I created a custom rule to generate a high-priority alert whenever PowerShell was executed.

_**[Screenshot 2: The Level 12 Sysmon alert for PowerShell execution]**_

### 2. Proactive Vulnerability Detection

I configured and enabled Wazuh's Vulnerability Detector module to proactively scan all endpoints for known Common Vulnerabilities and Exposures (CVEs). This demonstrates the ability to identify and prioritize system weaknesses before they can be exploited.

_**[Screenshot 3: The Vulnerabilities tab for the Windows agent, showing High/Medium CVEs]**_

### 3. Real-Time File Integrity Monitoring (FIM)

To protect critical assets, I configured FIM to monitor the web root directory (`/var/www/html`) on the Linux web server. This provides instant alerts for any unauthorized file modifications, such as a website defacement.

_**[Screenshot 4: The Level 7 "Integrity checksum changed" alert]**_

### 4. Threat Intelligence Integration with VirusTotal

I integrated the Wazuh server with the VirusTotal API. This automatically checks the hash of any new file detected by FIM against VirusTotal's database of known malware, turning a simple file event into a critical, high-fidelity malware detection.

_**[Screenshot 5: The Level 12 VirusTotal alert showing "malicious file detected"]**_

---

## Grand Finale: The Automated Kill Chain

This final demonstration showcases a complete, automated "Detect and Respond" workflow.

**The Scenario:** An attacker (Kali Linux) successfully compromises the Linux web server and attempts to escalate their attack by launching a brute-force SSH attack against other machines.

**1. The Attack (Before):** The attacker has full connectivity and can ping the target machine successfully.
_**[Screenshot 6A: The successful `ping` from Kali]**_

**2. Detection and Response:** Wazuh detects the multiple failed login attempts, triggering a Level 10 alert. This alert immediately triggers a pre-configured Active Response rule, which executes a script on the target agent to block the attacker's source IP address.
_**[Screenshot 6B: The Wazuh dashboard showing the Level 10 brute-force alert followed by the Level 3 "Host Blocked" alert]**_

**3. The Result (After):** The attacker's IP is now blocked at the firewall level. All further connection attempts from the attacker fail.
_**[Screenshot 6C: The failed `ping` from Kali, showing "Destination host unreachable"]**_

This successful demonstration proves the lab's capability to not only detect threats in real-time but to automatically neutralize them without human intervention.
