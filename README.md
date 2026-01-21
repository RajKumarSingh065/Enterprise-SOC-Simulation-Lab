# Enterprise SOC Automation Lab (Wazuh + Docker)

## Project Overview

This project simulates a modern Security Operations Center (SOC) environment using a **containerized Wazuh SIEM** architecture. Designed to operate within a resource-constrained environment (16GB RAM), it utilizes **Docker** for orchestration and a custom **TCP Tunneling** network configuration to ingest telemetry from isolated Windows endpoints.

The lab demonstrates a full security lifecycle: **Log Ingestion → Detection Engineering → Threat Intelligence Enrichment → Automated Response**.

---

## Architecture & Network Design

Unlike standard cloud deployments, this lab solves the challenge of running an enterprise-grade SIEM on a single host without bridging capabilities.

* **SIEM Core:** Wazuh Manager, Indexer, and Dashboard running as **Docker containers** on an Ubuntu VM.
* **Endpoint:** Windows 11 Host running the **Wazuh Agent** and **Sysmon** for deep visibility.
* **The Network Bridge:** Implemented a custom **NAT Port-Forwarding Tunnel** to route encrypted TCP traffic between the host and the containerized manager, bypassing standard isolation layers.

---

## Key Capabilities

### 1. Endpoint Detection & Response (EDR)

* Configured **Sysmon** to provide granular visibility into process creation, network connections, and DNS queries.
* Developed custom XML decoders to map "Living off the Land" binary (LOLBins) execution to **MITRE ATT&CK** techniques.

### 2. File Integrity Monitoring (FIM)

* Implemented real-time monitoring of sensitive directories (`C:\Sensitive` and `System32`).
* Configured alerts to trigger immediately upon file creation, modification, or deletion, capturing user identity and content "diffs."

### 3. Automated Threat Intelligence (VirusTotal)

* Integrated the **VirusTotal API** into the Wazuh detection engine.
* **Workflow:**
1. New file detected on endpoint.
2. Hash extracted and sent to VirusTotal.
3. If malicious (Positive match > 0), a high-severity alert is generated instantly.



### 4. Active Response (SOAR)

* Engineered an automated containment playbook using `active-response` scripts.
* **Scenario:** Upon detecting a high-severity FIM alert (e.g., ransomware behavior), the system automatically executes a `netsh` command on the victim machine to block network traffic, reducing **Mean Time to Respond (MTTR)** to <5 seconds.

---

## Tech Stack

* **SIEM:** Wazuh 4.x
* **Containerization:** Docker & Docker Compose
* **OS:** Ubuntu Server 22.04 (VM), Windows 11 (Host)
* **Telemetry:** Sysmon, Windows Event Logs
* **APIs:** VirusTotal
* **Networking:** TCP/IP, NAT, Port Forwarding

---

## Screenshots



## 🔧 Challenges Solved

**The Network Isolation Issue:**
Running the SIEM in Docker inside a NAT-ed VM prevented the external Windows Agent from connecting via UDP.

* **Solution:** Reconfigured the Wazuh Manager and Agent to communicate over **TCP Port 1514**. Established a persistent port-forwarding rule set (`127.0.0.1:1514` -> `10.0.2.15:1514`) to successfully tunnel log data through the virtualization layer.

---

