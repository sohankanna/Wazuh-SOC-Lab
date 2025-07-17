# Wazuh SIEM/XDR Home Lab Project

<p align="center">
  <img src="YOUR_ARCHITECTURE_DIAGRAM_URL_HERE" width="700">
</p>

## Overview

This project demonstrates the deployment and configuration of a comprehensive Security Operations Center (SOC) using the open-source Wazuh SIEM/XDR platform. The goal was to build a multi-machine virtual lab to simulate real-world security scenarios, including proactive vulnerability scanning, advanced threat detection, and automated incident response. This project involved extensive troubleshooting of networking, service configurations, and rule logic to create a stable, fully functional security monitoring environment.

---

## Lab Architecture

The lab was built using Oracle VirtualBox and consists of four virtual machines on a dual-homed network (a `NAT Network` for inter-VM communication and a `Host-Only` network for management):

- **Wazuh Server:** Ubuntu Server running the central Wazuh manager, indexer, and dashboard.
- **Windows Endpoint:** A Windows 11 Pro machine acting as a corporate workstation.
- **Linux Endpoint:** An Ubuntu Server, acting as a company server.
- **Attacker Machine:** A Kali Linux instance used to simulate threats.

_<img width="1914" height="983" alt="Screenshot 2025-07-17 121401" src="https://github.com/user-attachments/assets/8d26e274-d911-475e-b399-84752d61daf1" />


---

## Core Capabilities Demonstrated

### 1. Advanced Endpoint Monitoring with Sysmon

I enhanced endpoint visibility on the Windows agent by integrating Sysmon. This allowed for the detection of granular system activity often missed by standard logging. To prove the entire pipeline—from event generation on the agent to analysis on the server—was working, I created a custom rule to generate a high-priority alert whenever PowerShell was executed.

_**[Screenshot 2: The Level 12 Sysmon alert for PowerShell execution]**_

### 2. Proactive Vulnerability Detection

I configured and enabled Wazuh's Vulnerability Detector module to proactively scan all endpoints for known Common Vulnerabilities and Exposures (CVEs). The system successfully identified 61 vulnerabilities on the Windows 11 endpoint, demonstrating the ability to assess security posture and prioritize patching efforts based on severity.

_**[Screenshot 3: The Vulnerabilities tab for the Windows agent, showing High/Medium CVEs]**_

### 3. Real-Time File Integrity Monitoring (FIM)

To protect critical assets, I configured File Integrity Monitoring (FIM) to provide instant alerts for unauthorized file modifications. After initial troubleshooting revealed a persistent configuration sync issue, I successfully pivoted to a direct, local configuration on the agent. The test was conducted by creating a new file in the monitored `/home/arthur` directory, which immediately triggered the FIM alert. This demonstrated not only the ability to configure FIM but also to adapt and solve complex, real-world troubleshooting challenges.

_**[Screenshot 4: The Level 5 "File added to the system" alert from the Linux agent]**_

### 4. Threat Intelligence Integration with VirusTotal

I integrated the Wazuh server with the VirusTotal API. This automatically checks the hash of any new file detected by FIM against VirusTotal's database of known malware. A test using the EICAR file successfully triggered the FIM-to-VirusTotal pipeline, generating an alert that confirmed the successful API query and response. This turns a simple file event into an enriched, high-fidelity security signal.

_**[Screenshot 5: The VirusTotal alert showing "61 engines detected this file"]**_

---

## Grand Finale: The Automated Kill Chain

This final demonstration showcases a complete, automated "Detect and Respond" workflow using a classic brute-force attack scenario.

**The Scenario:** An attacker (Kali Linux) attempts to gain access to the `Linux-WebServer` by launching an SSH brute-force attack.

**1. The Attack (Before):** The attacker has full connectivity and can successfully `ping` the target machine.
_**[Screenshot 6A: The successful `ping` from Kali to the Linux server]**_

**2. Detection and Response:** Wazuh's log analysis engine detects the multiple failed SSH login attempts from the same source IP, triggering a Level 10 alert (Rule ID `2502`). This alert immediately triggers a pre-configured Active Response rule, which executes the `firewall-drop` script on the target agent to block the attacker's IP address.
_**[Screenshot 6B: The Wazuh dashboard showing the Level 10 brute-force alert followed by the Level 3 "Host Blocked" alert]**_

**3. The Result (After):** The attacker's IP is now blocked at the firewall level on the `Linux-WebServer`. All further connection attempts from the attacker fail.
_**[Screenshot 6C: The failed `ping` from Kali, showing "Destination host unreachable"]**_

This successful demonstration proves the lab's capability to not only detect threats in real-time but to automatically neutralize them without human intervention, completing the full security lifecycle.
