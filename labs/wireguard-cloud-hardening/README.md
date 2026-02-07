```markdown
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

```
