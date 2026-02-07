
# Lab: Secure SOC Architecture (WireGuard, Bastion, & Wazuh Pipeline)
**Focus:** Infrastructure-as-Code, Detection Engineering, and Private Telemetry

## 1. Project Overview
This lab demonstrates a multi-tier security architecture hosted on **Vultr**. It combines a secure remote access layer with a real-time detection pipeline. I deployed a Windows Server 2022 instance to act as both a **Bastion Host** and a **Logging Source**, feeding telemetry to an isolated Wazuh SIEM.

## 2. The Architecture
* **The Bastion (Windows Server 2022):** * Secured via **WireGuard VPN** (No public RDP).
    * Acting as a jump box to the internal network.
* **The Pipeline (Sysmon + Wazuh Agent):**
    * Configured **Sysmon** on the Windows host to capture granular process and network events.
    * Installed the **Wazuh Agent**, configured to ship logs over the **Vultr Private VPC** to the manager.
* **The SIEM (Wazuh Manager):**
    * Fully isolated on a separate Vultr instance with **no public IP**.
    * Only accessible via the Windows Bastion or the private internal network.

## 3. Technical Implementation
* **Cloud Provider:** Vultr
* **Telemetry:** **Sysmon** (Event ID 1, 3, and 22 focus) + **Wazuh Agent**.
* **Networking:** * **WireGuard:** Encrypted administrative tunnel.
    * **Private VPC:** 10Gbps backend link for log shipping (Zero exposure to the public internet).
* **Hardening:** Windows Firewall rules allow Wazuh traffic (Port 1514/1515) only via the internal interface.

## 4. Detection Capabilities
By connecting the Windows Server as a client to the Wazuh Manager, I successfully monitored:
* **Brute Force Attempts:** (Though neutralized by WireGuard, tested via internal simulation).
* **Process Anomalies:** Captured via Sysmon and alerted through Wazuh’s SCA (Security Configuration Assessment) and custom decoders.
* **Network Connections:** Monitored suspicious outbound traffic at the kernel level.

## 5. Security Analysis
This architecture follows the **Principle of Least Privilege**. By separating the "Management" (Wazuh) from the "Public Edge" (Windows), I ensured that even if a vulnerability were discovered in the Wazuh dashboard, it remains inaccessible to external attackers.

### 🛠️ Tech Stack
* **WireGuard** (Secure Access)
* **Wazuh** (SIEM/XDR)
* **Sysmon** (Enhanced Windows Logging)
* **Vultr VPC** (Network Segmentation)

## 6. Adversarial Emulation & Detection Testing
To verify the integrity of the detection pipeline, I simulated a common credential theft technique used by real-world attackers.

### Attack Scenario: OS Credential Dumping
* **Technique:** [MITRE ATT&CK T1003 (OS Credential Dumping)](https://attack.mitre.org/techniques/T1003/)
* **Execution:** Executed **Mimikatz** (`lsadump::sam`) on the Windows Server 2022 instance to attempt local SAM hash extraction.
* **The Goal:** To trigger a high-fidelity alert in Wazuh and verify that the Sysmon telemetry was correctly mapped.

![Mimikatz Execution](assets/mimikatz-execution.png)
*Figure 1: Mimikatz running on the Windows Server 2022 Bastion host, successfully extracting NTLM hashes.*

### Detection Logic & Custom Tuning
* **Custom Rule Escalation:** I manually tuned the Wazuh `ossec` rules to escalate this specific detection to **Level 15 (Extremely High Severity)**.
* **The Rationale:** In a production environment, credential dumping is a "critical stop" event. By increasing the severity from the default, I ensured that this alert stands out from lower-level system noise and demands immediate analyst response.
* **Result:** The system successfully captured the `lsass.exe` process access via Sysmon Event ID 10 and generated a Level 15 alert within seconds of execution.

TODO
![Wazuh Level 15 Alert](assets/wazuh-alert.jpg)
*Figure 2: Wazuh dashboard displaying the Level 15 alert for Mimikatz, confirming the success of the detection pipeline.*

