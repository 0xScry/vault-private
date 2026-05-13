# 🗺️ Network Enumeration with Nmap

**Path:** CPTS → Network Enumeration with Nmap **Tags:** #nmap #enumeration #recon #CPTS

---

## 📌 Methodology (Follow This Order Every Time)

```
1. Host Discovery       → Who is alive on the network?
2. Port Scanning        → What ports are open?
3. Service Enumeration  → What is running + what version?
4. OS Detection         → What OS is the target?
5. NSE Scripts          → Deeper enum / known vulns?
6. Save Output          → Always. Every scan.
```

> [!important] Always store every scan result. Different tools produce different results — having saved output lets you compare, document, and report accurately.

---

## 1. Host Discovery

**Goal:** Identify which hosts are alive before wasting time scanning dead ones.

### Scan a full subnet (ping sweep)

```bash
sudo nmap <target>/24 -sn -oA hosts_alive
```

### Scan from a list of IPs

```bash
sudo nmap -sn -oA hosts_alive -iL hosts.lst
```

> Useful in internal pentests where the client provides a scope list. Note: hosts that block ICMP won't appear — not necessarily dead.

### Scan multiple IPs / a range

```bash
sudo nmap -sn -oA hosts_alive 10.129.2.18 10.129.2.19 10.129.2.20
sudo nmap -sn -oA hosts_alive 10.129.2.18-20
```

### Scan a single host

```bash
sudo nmap <target> -sn -oA host
```

### Force ICMP Echo Requests (bypass ARP)

```bash
sudo nmap <target> -sn -PE --disable-arp-ping -oA host
```

> By default on LAN, Nmap uses ARP instead of ICMP. Use `--disable-arp-ping` to force ICMP Echo Requests.

### Diagnostic flags

```bash
--packet-trace    # show every packet sent/received
--reason          # show WHY nmap marked a host up/down
```

---

## 2. Port Scanning

### Port States

|State|Meaning|
|---|---|
|`open`|Connection established (TCP/UDP/SCTP)|
|`closed`|Port reachable but no service — returns RST flag|
|`filtered`|No response or error code — firewall likely dropping|
|`unfiltered`|Port accessible but open/closed can't be determined (ACK scan only)|
|`open\|filtered`|No response — firewall may be filtering|
|`closed\|filtered`|Only in IP ID idle scans|

---

### Scan Types

|Flag|Type|Notes|
|---|---|---|
|`-sS`|TCP SYN (Half-open)|Default as root. Sends SYN, reads SYN-ACK or RST. Never completes handshake → stealthier|
|`-sT`|TCP Connect|Default without root. Full 3-way handshake. More accurate, louder, logged by most systems|
|`-sU`|UDP Scan|Stateless, no handshake. Very slow. Critical for DNS/SNMP/TFTP|
|`-sN`|TCP Null|No flags set. FW evasion|
|`-sF`|TCP FIN|FIN flag only. FW evasion|
|`-sX`|TCP Xmas|FIN+PSH+URG flags. FW evasion|

> [!tip] SYN vs Connect
> 
> - **SYN (-sS):** Doesn't complete handshake → less logging → stealthier. Needs root.
> - **Connect (-sT):** Full handshake → creates logs → detected by IDS/IPS → but "polite", less likely to crash services.

---

### Port Ranges

```bash
nmap -p 22                   # single port
nmap -p 22,80,443            # multiple ports
nmap -p 1-1000               # range
nmap -p-                     # all 65535 ports
nmap -F                      # fast scan: top 100 ports
nmap --top-ports 10          # top 10 most common ports
nmap --top-ports 1000        # top 1000 (default behavior)
```

---

### Filtered Ports — Dropped vs Rejected

**Dropped (firewall silently drops packets):**

- Nmap gets no response → retries up to `--max-retries` (default: 10)
- Scan takes much longer (~2s per port)
- Port marked: `filtered`

**Rejected (firewall sends ICMP unreachable):**

- Nmap receives ICMP type=3 code=3 → port unreachable
- Scan is fast (immediate ICMP reply)
- Port still marked: `filtered`

```bash
# Trace a filtered port to see what's happening
sudo nmap <target> -p <port> --packet-trace -Pn -n --disable-arp-ping
```

---

### UDP Scanning

```bash
sudo nmap <target> -sU -F
```

**How Nmap interprets UDP responses:**

|Response|Port State|
|---|---|
|UDP reply received|`open`|
|No response (timeout)|`open\|filtered`|
|ICMP type=3 code=3 (port unreachable)|`closed`|
|Any other ICMP error|`open\|filtered`|

> [!warning] UDP is **stateless** — no 3-way handshake. Nmap sends empty datagrams. UDP scans are **very slow** (top 100 ports ≈ 98s). Only scan when needed. Key UDP ports to always check: DNS (53), SNMP (161/162), TFTP (69), NTP (123)

---

## 3. Service & Version Enumeration

**Goal:** Identify exactly what software and version is running on open ports.

### Basic version scan

```bash
sudo nmap <target> -p- -sV
```

### Monitor scan progress

```bash
# Press [Space Bar] during scan to see live status
sudo nmap <target> -p- -sV --stats-every=5s    # auto status every 5s
sudo nmap <target> -p- -sV -v                  # verbose: shows open ports as found
sudo nmap <target> -p- -sV -vv                 # extra verbose
```

### Banner Grabbing — Manual

Nmap sometimes misses info in service banners. Grab manually with `nc`:

```bash
nc -nv <target> <port>
```

Monitor traffic simultaneously with `tcpdump`:

```bash
sudo tcpdump -i eth0 host <your_ip> and <target_ip>
```

> [!note] Why manually banner grab? Example: Nmap shows `Postfix smtpd` on port 25. Running `nc -nv <target> 25` shows: `220 inlane ESMTP Postfix (Ubuntu)` — reveals the **OS distro**. The extra info comes from the PSH-ACK packet that Nmap doesn't always display. The 3-way handshake is: SYN → SYN-ACK → ACK, then server sends PSH-ACK with banner data.

---

## 4. NSE — Nmap Scripting Engine

Scripts are written in Lua. There are 14 categories:

|Category|Description|
|---|---|
|`auth`|Test/find authentication credentials|
|`broadcast`|Host discovery via broadcast|
|`brute`|Brute-force login attempts|
|`default`|Run with `-sC`|
|`discovery`|Evaluate accessible services|
|`dos`|Test for DoS vulnerabilities ⚠️|
|`exploit`|Exploit known vulnerabilities ⚠️|
|`external`|Use external services for processing|
|`fuzzer`|Fuzz services for unexpected behavior|
|`intrusive`|May negatively affect target ⚠️|
|`malware`|Check for malware infections|
|`safe`|Non-destructive, safe scripts|
|`version`|Extend service detection|
|`vuln`|Identify specific vulnerabilities|

### Script Usage

```bash
# Run default scripts
sudo nmap <target> -sC

# Run a whole category
sudo nmap <target> --script <category>

# Run specific scripts (comma-separated)
sudo nmap <target> --script <script-name>,<script-name>

# Example: grab banner + list SMTP commands on port 25
sudo nmap <target> -p 25 --script banner,smtp-commands

# Aggressive scan: sV + OS + traceroute + default scripts
sudo nmap <target> -p 80 -A

# Vulnerability scan on a specific port
sudo nmap <target> -p 80 -sV --script vuln
```

> [!tip] What `-A` does `-A` = `-sV` (versions) + `-O` (OS detection) + `--traceroute` + `-sC` (default scripts) Reveals: service versions, OS guess, network hops, and common misconfigurations.

### What good NSE output looks like

```
# --script banner,smtp-commands on port 25:
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS...

# -A on port 80 (WordPress site):
|_http-generator: WordPress 5.3.4
|_http-title: blog.inlanefreight.com

# --script vuln on port 80:
| http-wordpress-users: Username found: admin
| vulners:
|   CVE-2019-0211   7.2   https://vulners.com/cve/CVE-2019-0211
```

---

## 5. Performance Tuning

### Timing Templates

|Template|Name|Use Case|
|---|---|---|
|`-T0`|paranoid|Max stealth, IDS evasion|
|`-T1`|sneaky|Slow, IDS evasion|
|`-T2`|polite|Low bandwidth usage|
|`-T3`|normal|**Default**|
|`-T4`|aggressive|Fast, good network assumed|
|`-T5`|insane|Fastest, may miss results|

> In HTB labs → use `-T4`. In real/stealth engagements → `-T1` or `-T2`.

### Fine-grained performance options

```bash
--min-rate 300               # send at least 300 packets/sec (whitebox only)
--max-retries 0              # don't retry dropped packets (faster, risky)
--max-retries 3              # balanced retry count
--initial-rtt-timeout 50ms   # start with shorter RTT timeout
--max-rtt-timeout 100ms      # cap RTT timeout (reduces scan time)
--min-parallelism <n>        # minimum parallel probes
```

> [!warning] Aggressive tuning (low retries, short timeouts) **can cause you to miss hosts and ports**. Example: setting `--max-retries 0` found 2 fewer open ports vs default. Balance speed vs accuracy depending on the engagement type.

---

## 6. Saving Output

> [!important] Use `-oA` on **every scan** without exception. You will need these files for reporting, diffing results, and re-referencing during the engagement.

### Output formats

|Flag|Format|Extension|
|---|---|---|
|`-oN`|Normal (human readable)|`.nmap`|
|`-oG`|Grepable|`.gnmap`|
|`-oX`|XML|`.xml`|
|`-oA`|All three at once|`.nmap` + `.gnmap` + `.xml`|

```bash
sudo nmap <target> -p- -oA scan_results
# Creates: scan_results.nmap, scan_results.gnmap, scan_results.xml
```

### Convert XML to HTML report

```bash
xsltproc scan_results.xml -o scan_results.html
```

> Opens in browser as a clean, structured report. Useful for client documentation.

---

## 7. Full Scan Workflow (Use This in Labs & Exams)

```bash
# Step 1: Host discovery
sudo nmap <subnet>/24 -sn -oA 01_hosts_alive

# Step 2: Quick port scan (fast overview)
sudo nmap <target> -sS --top-ports 1000 -Pn -n --open -T4 -oA 02_quick_scan

# Step 3: Full port scan (all 65535 ports)
sudo nmap <target> -sS -p- -Pn -n --open -T4 -oA 03_full_port_scan

# Step 4: Service + version on discovered ports
sudo nmap <target> -sV -p <open_ports> -Pn -n -oA 04_service_scan

# Step 5: Default scripts on discovered ports
sudo nmap <target> -sC -sV -p <open_ports> -Pn -n -oA 05_script_scan

# Step 6: Aggressive scan on interesting ports (quick all-in-one)
sudo nmap <target> -A -p <open_ports> -Pn -n -oA 06_aggressive_scan

# Step 7: Vuln scripts on specific services
sudo nmap <target> -p 80 --script vuln -sV -oA 07_vuln_scan

# Step 8: UDP scan (never skip this)
sudo nmap <target> -sU -F -Pn -n -oA 08_udp_scan
```

---

## 8. Quick Reference — All Important Flags

|Flag|Purpose|
|---|---|
|`-sS`|SYN scan (stealthy, needs root)|
|`-sT`|TCP Connect scan (no root needed)|
|`-sU`|UDP scan|
|`-sV`|Service/version detection|
|`-sC`|Default NSE scripts|
|`-A`|Aggressive: sV + OS + traceroute + sC|
|`-O`|OS detection only|
|`-p-`|Scan all 65535 ports|
|`-F`|Fast scan (top 100 ports)|
|`--top-ports <n>`|Scan top N most common ports|
|`--open`|Show only open ports in output|
|`-Pn`|Skip host discovery (treat all as up)|
|`-n`|No DNS resolution|
|`--disable-arp-ping`|Force ICMP instead of ARP on LAN|
|`-PE`|Use ICMP Echo for ping|
|`--packet-trace`|Show all packets sent/received|
|`--reason`|Show why port is in its state|
|`-v` / `-vv`|Verbose / extra verbose output|
|`--stats-every=5s`|Show scan progress every 5s|
|`-T<0-5>`|Timing template (0=paranoid, 5=insane)|
|`--min-rate <n>`|Minimum packets/sec to send|
|`--max-retries <n>`|Max retries for unresponsive ports|
|`--initial-rtt-timeout`|Starting RTT timeout value|
|`--max-rtt-timeout`|Max RTT timeout value|
|`-oA <name>`|Save output in all 3 formats|
|`-oN / -oG / -oX`|Save in normal / grepable / XML|
|`-iL <file>`|Read targets from file|
|`--script <name/cat>`|Run NSE scripts or category|