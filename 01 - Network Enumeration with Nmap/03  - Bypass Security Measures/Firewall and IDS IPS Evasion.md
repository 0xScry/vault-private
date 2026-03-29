# Firewall and IDS/IPS Evasion with Nmap

### 1. Identifying Firewall Rules

Firewalls protect networks by monitoring traffic and deciding to **pass, ignore (drop), or block (reject)** packets based on defined rules. Understanding how a firewall responds is critical for determining the correct evasion technique.

- **Dropped Packets:** The firewall ignores the packet and returns **no response**. Nmap labels these ports as `filtered`.
- **Rejected Packets:** The firewall sends an explicit response. TCP packets return an **RST** flag; ICMP packets return error codes (e.g., net/host/port unreachable).

### 2. Bypassing Stateless Firewalls (ACK Scan)

**When to use:** Use when standard SYN scans (`-sS`) or Connect scans (`-sT`) are blocked by a firewall that filters incoming SYN packets. **Why it works:** Firewalls often allow packets with the **ACK flag** because they cannot determine if the connection was originally established from the internal network.

**Operational Workflow:**

1. Perform a standard SYN scan to identify `filtered` ports.
2. Execute an ACK scan on the same ports.
3. **Analyze results:** If an ACK scan receives an **RST** flag from a port previously marked as `filtered`, the port is actually **unfiltered** (and likely open), meaning the firewall only blocks connection initiation (SYN).

```
sudo nmap <TARGET_IP> -p <PORT> -sA -Pn -n --disable-arp-ping --packet-trace
```

### 3. Disguising Scan Origin (Decoys)

**When to use:** Use when an **IPS** is likely to block your IP address or when administrators block specific subnets. **Why it works:** Nmap inserts random IP addresses into the IP header to hide the actual source of the scan.

**Operational Workflow:**

1. Select a number of random decoys.
2. Ensure decoys are **alive**; if decoys are dead, the target may become unreachable due to SYN-flooding security mechanisms.
3. The attack IP is randomly placed among the decoys to mask the true origin.

```
sudo nmap <TARGET_IP> -p <PORT> -sS -Pn -n --disable-arp-ping -D RND:5
```

### 4. Testing Subnet-Based Access (Source IP Spoofing)

**When to use:** Use when the target network only allows access to specific services from **trusted subnets** or specific source IPs. **Why it works:** By spoofing a trusted source IP, you can bypass filters that are configured to trust internal or specific external ranges.

```
sudo nmap <TARGET_IP> -n -Pn -p <PORT> -O -S <SPOOFED_IP> -e <INTERFACE>
```

_Note: This often requires specifying the network interface (e.g., `tun0`)._

### 5. Exploiting Trusted Ports (Source Port Manipulation)

**When to use:** Use when a firewall is configured to trust traffic originating from specific ports, such as **DNS (UDP/TCP 53)**. **Why it works:** Administrators often weaken IDS/IPS filters for "trusted" ports to ensure critical services like DNS resolution function correctly.

**Operational Workflow:**

1. Identify a port marked as `filtered` via standard scanning.
2. Rescan using a common trusted source port like 53.
3. If successful, use **Netcat** to establish a connection using the same trusted source port.

**Scan Command:**

```
sudo nmap <TARGET_IP> -p <PORT> -sS -Pn -n --source-port 53
```

**Connection Command:**

```
ncat -nv --source-port 53 <TARGET_IP> <PORT>
```

### Command Reference: Evasion Options

|Option|Description|Logic/Decision Point|
|:--|:--|:--|
|`-sA`|TCP ACK Scan|Bypasses stateless firewalls that only block SYN packets.|
|`-D RND:<NUMBER>`|Decoy Scan|Masks the attack IP among a set of random, live IP addresses.|
|`-S <IP>`|Source IP Spoofing|Tests if a target service is accessible from a different subnet.|
|`--source-port <PORT>`|Set Source Port|Bypasses firewalls that trust specific ports like 53 (DNS).|
|`-Pn`|Disable ICMP Echo|Prevents Nmap from being blocked by firewalls that drop ICMP requests.|
|`-n`|Disable DNS Resolution|Increases speed and reduces noise by avoiding reverse DNS queries.|
|`--packet-trace`|Show Packet Traffic|Essential for verifying which flags (SYN, ACK, RST, SA) are being returned.|

### Misconfigured Security Settings

|Setting|Attack Implication|
|:--|:--|
|**Stateless Filtering**|Allows `-sA` (ACK) packets to pass, revealing "unfiltered" ports that can be further targeted.|
|**Trusted Source Ports**|Allows an attacker to bypass the firewall by simply setting their source port to 53 or other common service ports.|
|**Subnet Trust**|Allows an attacker to access restricted services by spoofing an IP from an allowed range.|