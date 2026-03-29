# Host Discovery

The objective of host discovery is to identify online systems within a network range to define the active attack surface.

### Discovery Methodology

1. **Define the Scope:** Identify whether you are scanning a network range, a specific list of IPs, or individual targets.
2. **Initial Ping Sweep:** Use ICMP echo requests to determine if hosts are alive.
3. **Validate "Alive" Status:** If a host appears down, determine if firewalls are dropping ICMP packets or if Nmap is defaulting to ARP pings (local network).

### Host Discovery Command Reference

|Goal|Command|
|:--|:--|
|Scan network range (no port scan)|`sudo nmap <TARGET_IP>/24 -sn -oA <FILENAME>`|
|Scan from IP list|`sudo nmap -sn -oA <FILENAME> -iL <HOSTS_LIST>`|
|Scan specific IP range|`sudo nmap -sn -oA <FILENAME> <TARGET_IP_START>-<END>`|
|Force ICMP Echo Scan (Local)|`sudo nmap <TARGET_IP> -sn -PE --packet-trace --disable-arp-ping`|
|View reason for "Alive" state|`sudo nmap <TARGET_IP> -sn --reason`|

---

# Port Scanning & Analysis

Nmap identifies the status of services by analyzing how they respond to specific packets.

### Port State Interpretations

|State|Description|Attack Implication|
|:--|:--|:--|
|**Open**|Connection established (TCP, UDP, or SCTP).|Direct target for exploitation.|
|**Closed**|Received **RST** flag; no service is listening.|Confirms host is alive but port is inactive.|
|**Filtered**|No response or ICMP error (Type 3/Code 3); packet dropped/rejected by firewall.|Requires evasion or a different approach to probe.|
|**Unfiltered**|Occurs during TCP-ACK scan; port is accessible but state is unknown.|Used to map firewall rules.|
|**Open\|Filtered**|No response received; state is ambiguous.|

### TCP Scanning Techniques

- **SYN Scan (`-sS`):** The default for root users. It is faster and more stealthy because it never completes the three-way handshake (half-open scan), minimizing connection logs.
- **Connect Scan (`-sT`):** Completes the three-way handshake. Use when accuracy is a priority or when running without root privileges. **Warning:** This is easily detected by IDS/IPS as it establishes a full connection.

|Goal|Command|
|:--|:--|
|Default SYN scan (Top 1000 ports)|`sudo nmap -sS <TARGET_IP>`|
|Full TCP Connect scan|`sudo nmap -sT <TARGET_IP>`|
|Scan specific port range|`sudo nmap -p <PORT_START>-<PORT_END> <TARGET_IP>`|
|Scan all ports (1-65535)|`sudo nmap -p- <TARGET_IP>`|
|Fast scan (Top 100 ports)|`sudo nmap -F <TARGET_IP>`|
|Scan Top X ports|`sudo nmap --top-ports=<NUMBER> <TARGET_IP>`|

### UDP Scanning (`-sU`)

**UDP is stateless** and significantly slower than TCP because Nmap must wait for timeouts since there is no acknowledgement.

- **When to use:** Use when you suspect an administrator has filtered TCP but forgotten to filter UDP.
- **Limitation:** If no response is received, Nmap marks it `open|filtered` because it cannot tell if the packet arrived.

---

# Scan Performance & Optimization

Optimizing performance is critical for large networks or limited bandwidth.

### Optimization Parameters

|Option|Description|Use Case|
|:--|:--|:--|
|`-T <0-5>`|Timing templates (0=Slow/Paranoid, 5=Insane).|`-T 3` is default. `-T 4/5` increases speed but risks blocking.|
|`--initial-rtt-timeout`|Sets initial Round-Trip-Time.|Lowering this (e.g., 50ms) speeds up scans but may skip hosts.|
|`--max-retries`|Number of times Nmap retries a port.|Set to `0` for maximum speed in reliable networks.|
|`--min-rate <number>`|Minimum packets sent per second.|Use in **White-box** tests when bandwidth is known.|

---

# Service Enumeration Methodology

Determining exact application versions is essential for finding precise exploits.

### Enumeration Workflow

1. **Initial Overiew:** Perform a quick port scan (`-F`) to identify easy targets.
2. **Deep Discovery:** Run a full port scan (`-p-`) in the background.
3. **Service Detection:** Run `-sV` on identified open ports to determine versions and service details.
4. **Manual Verification:** If Nmap cannot identify a version, use **Banner Grabbing** manually.

### Service Enumeration Commands

|Goal|Command|
|:--|:--|
|Service version detection|`sudo nmap -sV <TARGET_IP>`|
|Manual banner grab|`nc -nv <TARGET_IP> <PORT>`|
|Increase verbosity (see ports as found)|`sudo nmap -sV -v <TARGET_IP>`|
|Track progress every 5 seconds|`sudo nmap -p- --stats-every=5s <TARGET_IP>`|

---

# Nmap Scripting Engine (NSE)

NSE automates service interaction, vulnerability discovery, and brute-forcing.

### Common Script Categories

- **default (`-sC`):** Standard safe scripts.
- **auth:** Check for authentication credentials.
- **vuln:** Identify specific known vulnerabilities.
- **intrusive:** Scripts that might crash services or negatively affect the target.

### NSE & Aggressive Scanning

|Goal|Command|
|:--|:--|
|Aggressive Scan (Version, OS, Scripts, Traceroute)|`sudo nmap -A <TARGET_IP>`|
|Run default scripts|`sudo nmap -sC <TARGET_IP>`|
|Run specific scripts|`sudo nmap --script <SCRIPT_NAME1>,<SCRIPT_NAME2> <TARGET_IP>`|
|Scan for vulnerabilities|`sudo nmap -sV --script vuln <TARGET_IP>`|

---

# Result Management

Always save scan results for documentation and comparison.

### Output Formats

|Format|Flag|Description|
|:--|:--|:--|
|**All Formats**|`-oA <FILENAME>`|Saves in Normal, Grepable, and XML.|
|**Normal**|`-oN <FILENAME>`|Standard CLI output.|
|**Grepable**|`-oG <FILENAME>`|Easy to filter with command-line tools.|
|**XML**|`-oX <FILENAME>`|Required for technical documentation and HTML reports.|

### Operational Workflow: HTML Reporting

1. **Generate XML:** Run Nmap with `-oX` or `-oA`.
2. **Convert to HTML:** Use `xsltproc` to create a readable browser report for non-technical stakeholders.
    
    ```
    xsltproc <FILENAME>.xml -o <FILENAME>.html
    ```
    

---

# Security Misconfigurations Table

|Misconfiguration|Risk|Attack Implication|
|:--|:--|:--|
|**Unfiltered UDP**|Service Exposure|Admins often secure TCP but leave UDP open for services like DNS or SNMP.|
|**Detailed Service Banners**|Information Leakage|Banners can reveal OS versions (e.g., Ubuntu) that Nmap might not capture automatically.|
|**Insecure Timing Templates**|Blocking|Using `-T 5` can trigger security mechanisms and result in your IP being blocked.|