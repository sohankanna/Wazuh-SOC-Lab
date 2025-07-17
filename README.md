# Wazuh SIEM/XDR Home Lab Project

<p align="center">
  <img width="881" height="981" alt="Untitled Diagram drawio(1)" src="https://github.com/user-attachments/assets/36df27f3-0417-41a4-b6d7-10e022b91442" />

</p>

## Overview

**This project demonstrates the end-to-end deployment and configuration of a comprehensive Security Operations Center (SOC) using the open-source Wazuh SIEM/XDR platform. The goal was to build a multi-machine virtual lab to simulate and defend against real-world security threats. This involved extensive troubleshooting of networking, service configurations, and rule logic to create a stable, fully functional security monitoring environment that follows the full security lifecycle: from proactive defense to real-time detection and automated incident response.**

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

I enhanced endpoint visibility on the Windows agent by integrating Sysmon. This allowed for the detection of granular system activity often missed by standard logging. To simulate a high-fidelity detection for a common threat vector, I authored a custom rule to generate a high-priority alert whenever PowerShell was executed.

<details>
  <summary><b>View Custom Rule (local_rules.xml)</b></summary>

  ```xml
  <!-- Custom rule to force alert on PowerShell start -->
  <group name="sysmon,">
   <rule id="100500" level="12">
     <if_group>sysmon_event1</if_group>
     <field name="win.eventdata.image" type="pcre2">(?i)powershell\.exe</field>
     <description>Sysmon - Suspicious Process: PowerShell execution detected</description>
     <group>pci_dss_10.6.1,pci_dss_11.4,gdpr_IV_35.7.d,nist_800_53_AU.6,nist_800_53_SI.4,tsc_CC7.2,tsc_CC7.3,</group>
   </rule>
  </group>


```
</details>
<img width="1918" height="541" alt="Screenshot 2025-07-16 134621" src="https://github.com/user-attachments/assets/e5986be9-b7f3-442f-a962-c16add5d20b3" />

### 2. Proactive Vulnerability Detection

To protect critical assets, I configured File Integrity Monitoring (FIM) to provide instant alerts for unauthorized file modifications. After initial troubleshooting revealed a persistent configuration sync issue, I successfully pivoted to a direct, local configuration on the agent's `ossec.conf` file to monitor the user's home directory.

<details>
  <summary><b>View FIM Configuration (Linux Agent ossec.conf)</b></summary>

  ```xml
  <!-- FIM - Real-time monitoring of user's home directory -->
  <syscheck>
    <directories check_all="yes" realtime="yes">/home/arthur</directories>
  </syscheck>
```
</details>
<img width="1913" height="900" alt="Screenshot 2025-07-16 205949" src="https://github.com/user-attachments/assets/59a4500c-ce88-4d30-9c79-9ae723ed0cf0" />


### 3. Real-Time File Integrity Monitoring (FIM)

To protect critical assets, I configured File Integrity Monitoring (FIM) to provide instant alerts for unauthorized file modifications. After initial troubleshooting revealed a persistent configuration sync issue, I successfully pivoted to a direct, local configuration on the agent. The test was conducted by creating a new file in the monitored `/home/arthur` directory, which immediately triggered the FIM alert. This demonstrated not only the ability to configure FIM but also to adapt and solve complex, real-world troubleshooting challenges.

<img width="1905" height="828" alt="Screenshot 2025-07-16 230139" src="https://github.com/user-attachments/assets/db5774e6-4436-46cd-8e4e-39736c6051c5" />


### 4. Threat Intelligence Integration with VirusTotal
I integrated the Wazuh server with the VirusTotal API. This automatically checks the hash of any new file detected by FIM against VirusTotal's database of known malware, transforming a low-context FIM alert into an actionable, high-confidence security event.

<details>
  <summary><b>View VirusTotal Integration (Server ossec.conf)</b></summary>

  ```xml
  <!-- VirusTotal Integration -->
  <integration>
    <name>virustotal</name>
    <api_key>YOUR_API_KEY_HERE</api_key> <!-- API key hidden for security -->
    <group>syscheck</group>
    <alert_format>json</alert_format>
  </integration>

```
</details>
<img width="1913" height="846" alt="Screenshot 2025-07-17 012737" src="https://github.com/user-attachments/assets/4bff9a95-a9fa-4bd0-9df3-a0c9242776f9" />

---

## Grand Finale: The Automated Kill Chain


This final demonstration showcases a complete, automated "Detect and Respond" workflow using a classic brute-force attack scenario. I configured an Active Response to automatically block the source IP of an attacker after multiple failed SSH login attempts.

<details>
  <summary><b>View Active Response Configuration (Server ossec.conf)</b></summary>

  ```xml
  <!-- Active Response - Block IP on multiple failed logins -->
  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>2502</rules_id>
    <timeout>600</timeout>
  </active-response>
```
</details>

**The Scenario:** An attacker (Kali Linux) attempts to gain access to the `Linux-WebServer` by launching an SSH brute-force attack.

**1. The Attack (Before):** The attacker has full connectivity and can successfully `ping` the target machine.


<img width="640" height="295" alt="Screenshot 2025-07-17 010335" src="https://github.com/user-attachments/assets/58b0e28d-079f-4e9b-ab30-8c02474a0bec" />


**2. Detection and Response:** Wazuh's log analysis engine detects the multiple failed SSH login attempts from the same source IP, triggering a Level 10 alert (Rule ID `2502`). This alert immediately triggers a pre-configured Active Response rule, which executes the `firewall-drop` script on the target agent to block the attacker's IP address.

<img width="1919" height="861" alt="Screenshot 2025-07-17 015148" src="https://github.com/user-attachments/assets/8677175a-a7e1-4a75-97ed-a936febe0ccc" />

<img width="1904" height="708" alt="Screenshot 2025-07-17 014614" src="https://github.com/user-attachments/assets/65ea18c5-c749-4fda-99ca-623ac0b36b8e" />


**3. The Result (After):** The attacker's IP is now blocked at the firewall level on the `Linux-WebServer`. All further connection attempts from the attacker fail.

<img width="748" height="208" alt="Screenshot 2025-07-17 015648" src="https://github.com/user-attachments/assets/56e49162-8cce-45d9-bf13-4f072864c3be" />


This successful demonstration proves the lab's capability to not only detect threats in real-time but to automatically neutralize them without human intervention, completing the full security lifecycle.

## Future Improvements
This lab provides a strong foundation for further expansion. Future work could include:

  Integrating a SOAR platform like Shuffle or TheHive to automate ticketing and case management.

  Expanding to the cloud by deploying a Wazuh agent on an AWS EC2 instance.

  Developing more complex attack chains involving lateral movement and data exfiltration to test more advanced detection rules.
