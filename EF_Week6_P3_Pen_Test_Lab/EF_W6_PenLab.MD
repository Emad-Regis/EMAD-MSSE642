
***

# MSSE 642 - Hands-On Project No. 3

**Student:** Emad Fattah  
**Date:** June 2026  

---

## Part 1: Lab Setup — Both VMs Running and Connected

The lab environment consists of two virtual machines running in VirtualBox on a host-only network:
*   **Kali Linux** — the attacker machine
*   **Metasploitable2** — the intentionally vulnerable target machine (IP: 192.168.64.8)

##
![snapshot_1](/EF_Week6_P3_Pen_Test_Lab/P6_Images/Screenshot1.png)

##
Pic.1 (Two VMs running simultaneously in VirtualBox, with a successful ping from Kali Linux to Metasploitable2 (192.168.64.8) confirming network connectivity).

---

## Part 2: Metasploit Port Scanning

### Scan Process
Port scanning was performed using the Metasploit Framework's built-in TCP port scanner module. The following steps were executed from the Kali Linux terminal:

1. Initialized the Metasploit database and launched the console:
   ```bash
   sudo msfdb init && msfconsole
   ```
2. Searched for available port scanning modules:
   ```bash
   search portscan
   ```
3. Selected the TCP scanner:
   ```bash
   use auxiliary/scanner/portscan/tcp
   ```
4. Configured the target host:
   ```bash
   set RHOSTS 192.168.64.8
   ```
5. Restricted the scan to well-known ports: 
   ```bash
   set PORTS 1-1024
   ```
6. Executed the scan:
   ```bash
   run
   ```

![Snapshot_2](/EF_Week6_P3_Pen_Test_Lab/P6_Images/Screenshot2.png)
Pic.2 (Metasploit console running)

![Snapshot_3](/EF_Week6_P3_Pen_Test_Lab/P6_Images/Screenshot3.png)
Pic.3 (Port Scanner Modules (search portscan))

![Snapshot-4: Scan configuration (use, set RHOSTS, set PORTS)](image)

![Snapshot-5: Port Scan Results](image)

### Open Ports Identified
*   **Port 21:** FTP (File Transfer Protocol) — Used for transferring files.
*   **Port 22:** SSH (Secure Shell) — Used for secure remote server access.
*   **Port 23:** Telnet — Used for unencrypted, insecure remote access.
*   **Port 25:** SMTP (Simple Mail Transfer Protocol) — Used for sending email.
*   **Port 53:** DNS (Domain Name System) — Resolves domain names to IP addresses.
*   **Port 80:** HTTP (Hypertext Transfer Protocol) — Serves unencrypted web pages.
*   **Port 111:** RPCbind / Portmapper — Maps RPC services (often used by NFS).
*   **Port 139:** NetBIOS — Used for file and printer sharing on older Windows networks.
*   **Port 445:** SMB (Server Message Block) — Modern Windows file and printer sharing.
*   **Port 512:** rexec — Remote execution service (highly insecure).
*   **Port 513:** rlogin — Remote login service similar to Telnet (highly insecure).
*   **Port 514:** rsh (Remote Shell) or Syslog — Usually rsh in this context (insecure).

> **Note:** These open ports can be used by both Black Hat and White Hat Hackers.

---

### Port Scanning Q&A

#### Why do Black Hat Hackers use Port Scanning?
Black hat hackers use port scanning as a reconnaissance technique to map out a target's network architecture and identify viable entry points for exploitation. By sending packets to various logical ports on a system, attackers can determine which hosts are active, identify open digital doorways, and fingerprint the specific operating systems and services running on those ports. This allows them to discover vulnerable software versions and misconfigurations that can be matched against public exploit databases like CVE. Ultimately, port scanning transforms an unknown network into a detailed roadmap, enabling attackers to plan precise, targeted intrusions while minimizing time wasted on secure systems and reducing the likelihood of triggering security alarms.

#### How about White Hat Hackers?
White hat hackers use port scanning to proactively find and patch security weaknesses before malicious actors can exploit them. By scanning their own or a client's network, defensive security professionals map out active hosts and discover open digital ports to understand the organization's exact attack surface. This process allows them to identify unauthorized or forgotten services, detect dangerous misconfigurations, and verify that firewalls and access control policies are working as intended. Ultimately, white hats use the resulting data to audit system defenses, match open services against known CVE vulnerability databases, and apply the necessary patches or security controls to lock down the network.

#### Why is the scan restricted to Ports 1-1024?
Restricting a scan to ports 1–1024 allows security professionals to target a system's most critical core services while optimizing speed and evasion. This specific range contains the Well-Known Ports reserved for standard internet protocols and privileged operating system processes—such as [SSH](https://www.openssh.com/) (22), HTTP/HTTPS (80/443), and [SMB](https://www.samba.org/) (445)—which represent the highest-value targets for attackers because they often run with root or administrative privileges. By scanning fewer than 2% of the total 65,535 available ports, analysts drastically reduce network bandwidth consumption and compress scan times from hours to seconds. Furthermore, limiting the scope minimizes the noise generated across the network, reducing the likelihood of triggering rate-limiting firewalls, alerting Intrusion Detection Systems (IDS), or accidentally crashing fragile user applications hosted on higher ephemeral ports.

---

## Part 3: Comprehensive Report on the Nessus Vulnerability Assessment Tool

### 1. Introduction
In the modern cybersecurity landscape, vulnerability assessment tools play a critical role in identifying weaknesses before malicious actors can exploit them. Among these tools, Nessus stands out as one of the most widely adopted and trusted solutions for vulnerability scanning and security auditing. Developed by Tenable, Inc., Nessus has evolved from an open-source project into a professional-grade platform used by penetration testers, system administrators, and security auditors across the globe.

Nessus is designed to automate the discovery of vulnerabilities, misconfigurations, missing patches, and compliance issues across a wide range of systems and applications. Its extensive plugin library—updated daily—allows it to detect tens of thousands of known vulnerabilities, making it a cornerstone of proactive security management. The tool provides detailed reports, risk ratings, and remediation recommendations, enabling organizations to strengthen their security posture efficiently.

The official Nessus product page is available at:  
[https://www.tenable.com/products/nessus](https://www.tenable.com/products/nessus)

This report provides a detailed overview of Nessus, its purpose, its role in the penetration-testing lifecycle, and its significance as described in *Learn Kali Linux 2019* by Glen D. Singh.

### 2. Background and Evolution of Nessus
Nessus was originally created in 1998 by Renaud Deraison as an open-source vulnerability scanner. Over time, the project grew in popularity due to its accuracy, speed, and extensive vulnerability database. In 2005, the project transitioned from open-source to a commercial model under Tenable Network Security (now Tenable, Inc.). Despite this shift, Nessus remained accessible through a free “Home” version for personal use, while professional and enterprise versions expanded with advanced features.

Today, Nessus is part of Tenable’s broader ecosystem of exposure-management tools, including Tenable.io and Tenable.sc. However, Nessus remains the company’s flagship vulnerability scanner and is widely used in penetration testing, compliance auditing, and enterprise security operations.

### 3. Technical Overview of Nessus
Nessus operates using a client-server architecture. The **Nessus Server** performs the scanning, while the **Nessus Client**—typically accessed through a web interface—allows users to configure scans, review results, and manage reports.

#### Key Features
*   **Extensive Plugin Library:** Nessus uses over 170,000 plugins to detect vulnerabilities, each written in the Nessus Attack Scripting Language (NASL). These plugins are updated daily.
*   **Customizable Scan Templates:** Users can choose from templates such as:
    *   Basic Network Scan
    *   Credentialed Patch Audit
    *   Web Application Tests
    *   Malware Scan
    *   PCI DSS Compliance Scan
*   **Credentialed and Non-Credentialed Scanning:** Credentialed scans provide deeper insight into system configurations and missing patches.
*   **Comprehensive Reporting:** Reports include vulnerability descriptions, CVE references, CVSS scores, and remediation steps.
*   **Cross-Platform Support:** Nessus can scan Windows, Linux, macOS, network devices, databases, hypervisors, and cloud environments.

### 4. Nessus in the Penetration Testing Process
According to *Learn Kali Linux 2019* by Glen D. Singh, penetration testing follows a structured methodology consisting of several phases. Nessus plays a central role in the **Scanning and Vulnerability Assessment** phase, but its impact extends across the entire workflow.

#### 4.1 Information Gathering (Reconnaissance)
Before Nessus is used, penetration testers perform reconnaissance to identify:
*   Active hosts
*   Open ports
*   Running services
*   Operating systems
*   Network topology

Tools such as Nmap, Netdiscover, and DNS enumeration utilities are commonly used here. Nessus relies on this foundational information to perform targeted and efficient scans.

#### 4.2 Scanning and Vulnerability Assessment (Nessus’s Primary Role)
This is the phase where Nessus becomes indispensable. Singh emphasizes that Nessus is one of the most powerful tools available in Kali Linux for identifying vulnerabilities and misconfigurations.

Nessus performs:
*   Port scanning
*   Service enumeration
*   Patch auditing
*   Configuration assessment
*   Compliance checks
*   Credentialed vulnerability analysis

The tool assigns severity levels (Critical, High, Medium, Low, Informational) based on CVSS scoring, helping testers prioritize which vulnerabilities to investigate further.

#### 4.3 Analysis and Prioritization
After scanning, testers analyze Nessus reports to determine:
*   Which vulnerabilities are exploitable
*   Which weaknesses pose the highest risk
*   Which findings require manual verification

This step bridges automated scanning with human expertise.

#### 4.4 Exploitation
Nessus does **not** exploit vulnerabilities. Instead, it provides the intelligence needed to guide exploitation using tools such as:
*   Metasploit
*   Burp Suite
*   Hydra
*   Custom scripts

For example, if Nessus identifies an outdated SMB service, the tester may attempt an SMB exploit in Metasploit.

#### 4.5 Post-Exploitation and Reporting
While Nessus is not used in post-exploitation, its reports contribute to the final penetration-testing documentation. Nessus’s detailed remediation recommendations help organizations patch vulnerabilities and improve their security posture.

### 5. Advantages of Using Nessus
*   **5.1 Accuracy and Depth:** Nessus is known for its high detection accuracy due to its massive plugin database and frequent updates.
*   **5.2 Ease of Use:** Its web-based interface makes it accessible even to beginners, while still offering advanced customization for professionals.
*   **5.3 Comprehensive Coverage:** Nessus can scan operating systems, network devices, databases, virtual machines, cloud environments, and web applications.
*   **5.4 Strong Reporting Capabilities:** Reports are exportable in multiple formats and include vulnerability descriptions, CVE references, CVSS scores, and remediation steps.
*   **5.5 Integration with Security Programs:** Nessus integrates well with enterprise security workflows, compliance programs, and SIEM systems.

### 6. Limitations of Nessus
While Nessus is powerful, it has limitations:
*   It does not exploit vulnerabilities.
*   It may produce false positives that require manual verification.
*   Credentialed scans require administrative access, which may not always be available.
*   Some advanced features require paid licensing.

These limitations highlight the importance of combining Nessus with manual testing and other tools.

### 7. Lab — Running Nessus Against Metasploitable2
Nessus Installation and Usage on Kali Linux on the Kali Linux VM was built into the Project #1 lab setup, so no extra steps were needed. Once both virtual machines were running and connected, a Basic Network Scan targeted the Metasploitable2 IP.

The scan generated a detailed list of vulnerabilities across critical, high, medium, and low levels. A donut chart summarized the findings on the interface. The tool identified flaws in services like SMB, DNS, FTP, and RPC.

This highlights the value of a vulnerability scanner over a port scanner. While Metasploit simply found open ports, Tenable Nessus matched them with exact vulnerabilities, risk severity ratings, CVE references, and instructions to fix them. Nessus turned a list of open ports into a clear guide for security improvement.

### 8. Conclusion
Nessus is one of the most influential and widely used vulnerability-assessment tools in the cybersecurity industry. As described in *Learn Kali Linux 2019* by Glen D. Singh, Nessus plays a central role in the scanning and auditing phases of penetration testing, bridging the gap between reconnaissance and exploitation. Its extensive plugin library, customizable scan templates, and detailed reporting capabilities make it an essential tool for identifying and prioritizing security weaknesses.

By automating vulnerability detection and providing actionable remediation guidance, Nessus empowers organizations to strengthen their defenses proactively. Whether used by penetration testers, system administrators, or compliance auditors, Nessus remains a cornerstone of modern cybersecurity operations.

---

## References

*   Singh, G. D. (2019). *Learn Kali Linux 2019: Perform powerful penetration testing using Kali Linux, Metasploit, Nessus, Nmap, and Wireshark*. Packt Publishing.
*   Tenable, Inc. (2024). *Nessus vulnerability scanner*. Retrieved from [https://www.tenable.com/products/nessus](https://www.tenable.com/products/nessus)
*   Scarfone, K., & Mell, P. (2008). *Guide to penetration testing and vulnerability assessment* (NIST Special Publication 800-115). National Institute of Standards and Technology.
*   The Penetration Testing Execution Standard (PTES). (2024). *Technical guidelines*. Retrieved from [https://www.pentest-standard.org](https://www.pentest-standard.org/)
*   MITRE Corporation. (2024). *Common Vulnerabilities and Exposures (CVE)*. Retrieved from [https://cve.mitre.org](https://cve.mitre.org/)
