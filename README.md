# Cybersecurity: Snort IDS Configuration & Traffic Analysis

## 📖 Table of Contents
- [Introduction](#-introduction)
- [Objective](#-objective)
- [Lab Environment](#️-lab-environment)
- [Methodology & Practical Tasks](#-methodology--practical-tasks)
  - [1. Installation & Configuration](#1-installation--configuration)
  - [2. Writing Custom Rules](#2-writing-custom-rules)
  - [3. Initializing Snort in IDS Mode](#3-initializing-snort-in-ids-mode)
  - [4. Triggering & Analyzing Alerts](#4-triggering--analyzing-alerts)
- [Executive Summary & Conclusion](#-executive-summary--conclusion)
- [Ethical Guidelines & Disclaimer](#️-ethical-guidelines--disclaimer)

---

## Introduction
Snort is an industry-standard, open-source Network Intrusion Detection System (NIDS) and Intrusion Prevention System (NIPS). It allows security professionals and network administrators to monitor network traffic in real-time, analyze packets, detect suspicious or malicious activities, and generate alerts based on predefined or custom rules. 

A Snort rule is a text-based instruction that tells the engine exactly what type of network traffic to inspect and what action to take when a specific pattern is detected. These rules are processed sequentially to identify everything from basic reconnaissance to advanced exploitation attempts.

## 🎯 Objective
To deploy, configure, and operate Snort as a Network Intrusion Detection System (NIDS)[cite: 1]. The objective is to write custom detection signatures (rules) and validate their effectiveness by generating alerts for specific network anomalies, including ICMP sweeps, aggressive Port Scans, and SSH Brute Force attacks[cite: 1].

## 🛠️ Lab Environment
*   **Operating System:** Kali Linux (IDS Sensor & Attacker)
*   **Network Interface:** `eth0`
*   **Tool Used:** Snort 3 (or Snort 2.9.x)
*   **Configuration File:** `/etc/snort/snort.lua` (or `snort.conf`)
*   **Rule File:** `/etc/snort/rules/local.rules`

---

## Methodology & Practical Tasks

### 1. Installation & Configuration
**Objective:** To install the Snort engine and configure it to monitor the correct network interface[cite: 1].

**Methodology:**
Snort was installed via the terminal. Once installed, the primary configuration file (`snort.lua` or `snort.conf`) was modified to define the `HOME_NET` (the network being protected) and the `EXTERNAL_NET` (the untrusted outside world). 
*   **Command used for editing:** `sudo nano /etc/snort/snort.lua` (Saved using `Ctrl+O`, exited with `Ctrl+X`).

**Observation:**
Properly defining the `HOME_NET` prevents Snort from generating false positives on outbound traffic and focuses its processing power on inbound threats.

---

### 2. Writing Custom Rules
**Objective:** To create custom text-based instructions (`local.rules`) to detect specific attack vectors[cite: 1].

**Methodology:**
I edited the local rules file using `sudo nano /etc/snort/rules/local.rules` to create custom detection signatures. A Snort rule consists of a rule header (action, protocol, IPs, ports) and rule options (messages, exact payload matches, and SIDs).

**Rules Implemented:**
*   **ICMP Sweep Detection:**
    `alert icmp any any -> $HOME_NET any (msg:"ALERT: ICMP Ping Detected"; sid:1000001; rev:1;)`
*   **Nmap TCP Port Scan Detection:**
    `alert tcp any any -> $HOME_NET any (msg:"ALERT: Nmap TCP Scan Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 10; sid:1000002; rev:1;)`
*   **SSH Brute Force Detection:**
    `alert tcp any any -> $HOME_NET 22 (msg:"ALERT: SSH Brute Force Attempt"; flags:S; threshold:type both, track by_src, count 5, seconds 30; sid:1000003; rev:1;)`

**Result / Evidence:**
<br>

![Snort Custom Rules](images/snort-custom-rules.png)

---

### 3. Initializing Snort in IDS Mode
**Objective:** To launch Snort and bind it to the network interface for real-time packet processing.

**Command Executed:**
`sudo snort -c /etc/snort/snort.lua -R /etc/snort/rules/local.rules -i eth0 -A console`

**Command Breakdown:**
*   `sudo snort`: Runs the engine with elevated privileges required for packet sniffing.
*   `-c /etc/snort/snort.lua`: Points Snort to the main configuration file.
*   `-R /etc/snort/rules/local.rules`: Forces Snort to explicitly load our custom rules file.
*   `-i eth0`: Binds the listener to the `eth0` network interface.
*   `-A console`: Instructs Snort to print alerts directly to the terminal screen in real-time (Fast Alert mode).

**Result / Evidence:**
<br>

![Snort Initialization](images/snort-init.png)

---

### 4. Triggering & Analyzing Alerts
**Objective:** To validate the custom rules by simulating the attacks and monitoring the resulting alerts[cite: 1].

**Methodology:**
With Snort running in IDS mode on the console, I opened a second terminal window to act as the "Attacker." I executed the following actions to trigger the rules:
1.  **ICMP:** Sent ping requests to the IDS interface (`ping -c 4 [Target_IP]`).
2.  **Port Scan:** Ran a fast Nmap SYN scan (`nmap -sS -F [Target_IP]`).
3.  **SSH Brute Force:** Attempted rapid SSH logins or used a tool like Hydra to simulate password guessing.

**Observation:**
The Snort engine successfully intercepted the packets in real-time, matched the malicious traffic patterns against `local.rules`, and generated priority alerts directly to the console. The output clearly identified the source IPs, destination IPs, protocols, and the exact rule (SID) that was triggered.

**Result / Evidence:**
<br>

![Snort Alerts Firing](images/snort-alerts.png)

---

## Executive Summary & Conclusion
The Snort command is a powerful, flexible, and open-source network security tool that plays a vital role in protecting computer networks. In this lab, Snort was successfully configured from the ground up as a Network Intrusion Detection System (NIDS). 

By writing custom detection rules, the engine was able to successfully identify and alert on reconnaissance attempts (ICMP/Port Scanning) and active exploitation attempts (SSH Brute Forcing). Its real-time traffic analysis, customizable rule engine, and threat detection capabilities prove why it is so widely used in cybersecurity labs, enterprise networks, and Security Operations Centers (SOCs) globally.

---

## ⚖️ Ethical Guidelines & Disclaimer
This intrusion detection deployment and subsequent attack simulation were conducted entirely within a private, self-hosted Virtual Machine laboratory network. All traffic generation, scanning, and monitoring were performed strictly for educational and defensive cybersecurity training purposes.
