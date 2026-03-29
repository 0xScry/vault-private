
---
# Enumeration Methodology

**Enumeration** is the most critical phase of an assessment, focused on identifying all possible attack vectors rather than simply gaining access. The goal is to collect as much information as possible about services, protocols, and technologies to simplify finding vulnerabilities.

## Manual vs. Automated Enumeration

While scanning tools accelerate the process, they cannot always bypass security measures and should not replace active interaction with services.

### Automated Tool Limitations

- **Timeouts:** Most scanning tools have preset timeouts for service responses. If a service does not respond within that window, the port may be incorrectly marked as **closed**, **filtered**, or **unknown**.
- **False Negatives:** If a port is marked closed due to a timeout, a potential entry point may be overlooked.
- **Knowledge Gaps:** Tools are limited by their programming; manual interaction is required to understand how a service is meant to function and what information it provides.

## Key Attack Drivers

Attackers prioritize looking for the following two conditions during scanning and inspection:

1. **Misconfigurations:** Often resulting from ignorance or a poor security mindset (e.g., relying solely on firewalls or GPOs while neglecting service-level security).
2. **Neglect:** General lack of security maintenance for specific services.

---

# Nmap (Network Mapper)

**Nmap** is an open-source tool used for network analysis and security auditing. It identifies available hosts, services, application versions, and operating systems. It is also used to determine if **firewalls**, **packet filters**, or **Intrusion Detection Systems (IDS)** are configured correctly.

## Command Syntax

|Component|Description|
|:--|:--|
|`nmap`|The executable command|
|`<SCAN_TYPES>`|The specific technique used to probe the target|
|`<OPTIONS>`|Additional parameters for performance or output|
|`<TARGET_IP>`|The IP address or range to be scanned|

## Operational Techniques

### TCP-SYN Scan (-sS)

- **When to use:** This is the default scan method and is used when high-speed scanning is required (capable of scanning thousands of ports per second).
- **Mechanism:** It sends a packet with the **SYN** flag but **never completes the three-way handshake**. Because a full TCP connection is not established, it is often more efficient.

|Goal|Command|
|:--|:--|
|Perform a default stealth/SYN scan|`sudo nmap -sS <TARGET_IP>`|

### Analyzing Scan Results

Nmap output organizes data into three primary columns to help identify the attack surface:

- **PORT:** The port number and protocol (e.g., 22/tcp).
- **STATE:** The status of the service (e.g., **open**, **closed**, or **filtered**).
- **SERVICE:** The type of service running on that port (e.g., ssh, http, postgresql).