**Nmap — Host & Port Scanning**

**Port States:**

|State|Meaning|
|---|---|
|`open`|Connection established|
|`closed`|RST received|
|`filtered`|No response / error — firewall likely dropping|
|`unfiltered`|Accessible but open/closed can't be determined (ACK scan only)|
|`open\|filtered`|No response — firewall may be protecting it|
|`closed\|filtered`|IP ID idle scan only|

---

**Port Selection:**

```bash
-p 22,80,445       # specific ports
-p 22-445          # range
-p-                # all 65535
-F                 # top 100
--top-ports=10     # top N
```

**Scan Types:**

```bash
-sS   # SYN (default as root) — stealthy, no full handshake
-sT   # Connect (default without root) — full handshake, louder, more accurate
-sU   # UDP — slow, no handshake
-sV   # Version detection
```

**Filtered ports — dropped vs rejected:**

- **Dropped** → no response, Nmap retries 10x → slow (~2s)
- **Rejected** → ICMP type 3 code 3 (port unreachable) → fast response

---

**Service Version Detection:**

```bash
sudo nmap 10.129.2.28 -p- -sV
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s   # progress updates
sudo nmap 10.129.2.28 -p- -sV -v                  # live open port output
```

**Banner grabbing manually** (Nmap misses things):

```bash
nc -nv 10.129.2.28 25
# often reveals more than -sV (e.g. OS from SMTP banner)
```

---

**NSE (Nmap Scripting Engine):**

```bash
-sC                              # default scripts
--script <category>              # e.g. vuln, brute, auth
--script banner,smtp-commands    # specific scripts
-A                               # sV + OS + traceroute + sC
```

Key categories: `auth`, `brute`, `vuln`, `discovery`, `exploit`, `safe`

```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln   # CVE check
sudo nmap 10.129.2.28 -p 80 -A                  # aggressive all-in-one
```

---

**Output Formats:**

```bash
-oN   # normal (.nmap)
-oG   # grepable (.gnmap)
-oX   # XML (.xml)
-oA   # all three at once
```

```bash
xsltproc target.xml -o target.html   # convert XML → readable HTML report
```

---

**Performance Tuning:**

```bash
--min-rate 300              # send min 300 pkts/sec
--max-retries 0             # no retries (faster, miss more)
--initial-rtt-timeout 50ms
--max-rtt-timeout 100ms
```

**Timing templates** (`-T 0` slowest → `-T 5` fastest):

| Template | Name             |
| -------- | ---------------- |
| `-T0`    | paranoid         |
| `-T1`    | sneaky           |
| `-T2`    | polite           |
| `-T3`    | normal (default) |
| `-T4`    | aggressive       |
| `-T5`    | insane           |

> Faster ≠ better — aggressive timing can miss hosts and trigger IDS