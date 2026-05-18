1. SYN scan returns `filtered`.
2. Run ACK scan (`-sA`) to check if the firewall is stateful or filtering based on the SYN flag.
3. If ports show as `unfiltered`, use `--source-port 53` to exploit weak firewall rules trusting DNS traffic.
4. If scanning triggers an IDS/IPS block, switch to decoy scanning (`-D RND:<COUNT>`) or use different VPS IPs.
5. If specific subnets are suspected to have access, spoof the source IP (`-S`).

---

## Firewall Rule Identification

Standard SYN scans return `filtered` when the firewall drops packets without response or `closed` when it explicitly rejects them with RST/ICMP errors.

ACK scan to identify if the firewall passes packets regardless of state

```
sudo nmap <TARGET_IP> -p <PORT> -sA -Pn -n --disable-arp-ping --packet-trace
```

- **SYN Scan** → `-sS` → Prefer for initial discovery; identifies port state as open/closed/filtered.
    
- **ACK Scan** → `-sA` → Prefer for mapping firewall rules; identifies ports as `unfiltered` if they return an RST flag.
    
- **Open ports appearing as unfiltered**: The ACK scan cannot distinguish between open and closed ports; it only confirms the firewall allows the packet through.
    

## Source Port Manipulation

The firewall blocks standard high-port requests but allows traffic originating from "trusted" ports like DNS (53).

SYN scan from the DNS source port

```
sudo nmap <TARGET_IP> -p <PORT> -sS -Pn -n --disable-arp-ping --source-port 53
```

Direct service connection using a specific source port

```
ncat -nv --source-port 53 <TARGET_IP> <PORT>
```

- Misconfigured firewalls often trust TCP/UDP 53 to allow DNS resolution without inspecting the payload.
    
- **IDS/IPS detection**: Even if the firewall allows port 53, an active IPS may still inspect and block non-DNS traffic on that port.
    

## Source IP Spoofing

Access is restricted to specific trusted subnets or internal IP ranges.

Scan using a different source IP and specific interface

```
sudo nmap <TARGET_IP> -n -Pn -p <PORT> -O -S <PIVOT_IP> -e <SERVICE_NAME>
```

> ⚠️ Gap: Nmap cannot receive the response packets when spoofing an IP unless you have a way to capture traffic on the spoofed host's network or the target network.

- **ISP/Router egress filtering**: Most modern networks will drop packets with spoofed source IPs that do not belong to their range.

## Decoy Scanning

The target uses an active IDS/IPS that blocks the scanner's IP after a few probes.

Hide the real attack IP among five random alive decoys

```
sudo nmap <TARGET_IP> -p <PORT> -sS -Pn -n --disable-arp-ping -D RND:5
```

- **Dead decoys**: Using inactive IP addresses for decoys can lead to SYN-flooding the target, causing service outages or triggering aggressive security responses.

## Output Management

Scanning results need to be preserved for comparison or reporting.

Save results in normal, grepable, and XML formats simultaneously

```
sudo nmap <TARGET_IP> -p- -oA <FILE_PATH>
```

Convert XML output to a readable HTML report

```
xsltproc <FILE_PATH>.xml -o <FILE_PATH>.html
```

- **Missing paths**: If no full path is provided, Nmap writes files to the current working directory.