## Methodology

1. **Host Discovery**

- Sweep network range with `-sn` to identify active targets.
- Use `-PE` and `--disable-arp-ping` if local ARP responses are masking external firewall behavior.
- If ICMP is blocked, move to port discovery on common ports to confirm "alive" status.

2. **Surface Mapping**

- Fast scan top ports (`-F` or `--top-ports`) for quick wins.
- Full TCP port scan (`-p-`) in background for comprehensive coverage.
- Targeted UDP scan (`-sU`) for stateless services often missed by admins.

3. **Service Identification**

- Execute `-sV` on discovered open ports to pull versions and banners.
- Use `-A` for aggressive identification including OS and traceroute.
- Manually verify banners with `nc` if Nmap output is truncated or inconclusive.

4. **Vulnerability & Scripting**

- Run `--script vuln` against identified services to check for known CVEs.
- Use specific NSE categories (auth, brute, discovery) based on service type.

5. **Optimization**

- Adjust timing templates (`-T 0-5`) based on network stability and detection risk.
- Set `--min-rate` if network bandwidth is known and speed is priority.

---

## Host Discovery

Standard sweep when starting internal tests to identify live systems.

Identify active hosts in a CIDR range and output to all formats for grepping

```
sudo nmap <TARGET_IP>/24 -sn -oA tnet
```

Extract live IPs from greppable output

```
grep "Up" tnet.gnmap | cut -d" " -f2
```

Scan a specific list of targets provided in a file

```
sudo nmap -sn -oA tnet -iL hosts.lst
```

Force ICMP echo requests and disable ARP ping to see how external firewalls handle packets

```
sudo nmap <TARGET_IP> -sn -PE --packet-trace --disable-arp-ping
```

**Dangerous / misconfigured settings**

- Default ICMP may be ignored by host firewalls, leading to **false negatives** where hosts are marked inactive.

**Gotchas**

- **ARP responses override ICMP** on local networks, meaning Nmap will mark a host "up" before sending an ICMP request.

## TCP Port Scanning

Primary technique to identify established listener states.

SYN scan for stealth and speed on top 1000 ports

```
sudo nmap <TARGET_IP> -sS
```

TCP Connect scan when lacking root/socket permissions or requiring high accuracy

```
sudo nmap <TARGET_IP> -sT
```

Scan all 65535 ports to find non-standard listeners

```
sudo nmap <TARGET_IP> -p-
```

### Port State Decision Logic

- **open**: Three-way handshake completed or SYN-ACK received.
- **closed**: Received RST flag; host is alive but no service is listening.
- **filtered**: No response received (dropped) or ICMP unreachable type 3/code 3 (rejected).
- **unfiltered**: Only seen in ACK scans; port is accessible but state is unknown.

**Gotchas**

- **Root privileges required** for SYN scans; otherwise Nmap defaults to the noisier TCP Connect scan.

## UDP Port Scanning

Used to find stateless services like DNS, SNMP, or DHCP often overlooked by admins.

Fast UDP scan of top 100 ports

```
sudo nmap <TARGET_IP> -sU -F
```

Verify specific UDP port state with reasoning

```
sudo nmap <TARGET_IP> -sU -p <PORT> --reason
```

**Edge cases**

- UDP scans are significantly slower because Nmap must wait for timeouts on a stateless protocol.

**Gotchas**

- **Empty datagrams** sent by Nmap often trigger no response, resulting in an ambiguous `open|filtered` state.

## Service and Version Detection

Requirement for finding precise exploits and identifying the underlying OS.

Probe open ports for service versions

```
sudo nmap <TARGET_IP> -p <PORT> -sV
```

Aggressive scan combining versioning, OS detection, and default scripts

```
sudo nmap <TARGET_IP> -A
```

Increase verbosity to see discovered ports in real-time

```
sudo nmap <TARGET_IP> -p- -sV -v
```

Monitor scan progress at set intervals

```
sudo nmap <TARGET_IP> -p- -sV --stats-every=5s
```

### Manual Banner Grabbing

Use when Nmap fails to identify a version or the banner is manipulated.

Manual connection to grab service banner

```
nc -nv <TARGET_IP> <PORT>
```

Intercept traffic during connection to see raw PSH flags and identification data

```
sudo tcpdump -i eth0 host <ATTACK_IP> and <TARGET_IP>
```

**Gotchas**

- **Signature-based matching** used by `-sV` significantly increases scan duration compared to simple banner grabbing.

## Nmap Scripting Engine (NSE)

Automated interaction with services using Lua scripts.

Run default discovery and safety scripts

```
sudo nmap <TARGET_IP> -sC
```

Execute all scripts in a specific category like vulnerability identification

```
sudo nmap <TARGET_IP> --script vuln
```

Run specific scripts for service-specific enumeration

```
sudo nmap <TARGET_IP> -p 25 --script banner,smtp-commands
```

**Dangerous / misconfigured settings**

- `intrusive`: May crash services or negatively affect the target.
- `dos`: Designed to test denial of service vulnerabilities; high risk of service interruption.

**Gotchas**

- **Specific script arguments** (e.g., `http-wordpress-users.limit`) may be required to get complete results from NSE scripts.

## Performance Optimization

Necessary for large networks or restricted bandwidth scenarios.

Set aggressive timing template for fast networks

```
sudo nmap <TARGET_IP> -T 5
```

Increase packet rate to force speed on whitelisted targets

```
sudo nmap <TARGET_IP> --min-rate 300
```

Fine-tune timeouts for high-latency environments

```
sudo nmap <TARGET_IP> --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

Disable retries to maximize speed at the cost of accuracy

```
sudo nmap <TARGET_IP> --max-retries 0
```

**Dangerous / misconfigured settings**

- `-T 5`: Can be easily detected and blocked by IDS/IPS due to high traffic volume.

**Gotchas**

- **Over-optimized RTT timeouts** will cause Nmap to skip live hosts and open ports if responses are slightly delayed.

## Report Generation

Documentation of results for parsing or client delivery.

Save scan in Normal, Grepable, and XML formats simultaneously

```
sudo nmap <TARGET_IP> -oA <FILE_PATH>
```

Convert XML output to a readable HTML report

```
xsltproc <FILE_PATH>.xml -o <FILE_PATH>.html
```