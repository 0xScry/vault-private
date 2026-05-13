[MODE: full]

1. **Passive identification**: Listen for ARP, MDNS, and LLMNR traffic to map the **broadcast domain** without sending packets.
2. **Active validation**: Execute ICMP sweeps to confirm live hosts and build a target list.
3. **Service enumeration**: Fingerprint services (LDAP, Kerberos, SMB) and OS versions to identify **Domain Controllers** and legacy systems.
4. **Username harvesting**: Perform Kerberos pre-authentication checks to find valid domain accounts without triggering standard auth logs.

---

## Passive Network Monitoring

Blind entry or evasive requirements where active scanning is restricted; requires being on a switched network to capture **layer two packets**.

Run packet capture to identify MDNS and ARP traffic.

```
sudo tcpdump -i <SERVICE_NAME>
```

Identify hosts via LLMNR, NBT-NS, and MDNS without poisoning.

```
sudo responder -I <SERVICE_NAME> -A
```

- **Wireshark** → GUI → Prefer for deep packet inspection and visual traffic analysis.
- **tcpdump** → CLI → Prefer for headless hosts or saving **PCAP** files for remote analysis.
- **Responder** → Analyze Mode → Prefer for automated host and naming convention discovery.

**Gotchas**: **Switched Networks**. Capture is limited to the current **broadcast domain**; traffic from other segments will not be visible.

## Active Host Discovery

Validating passive results or mapping a CIDR range when ICMP is not blocked.

Perform a quiet round-robin ICMP sweep against a subnet.

```
fping -asgq <TARGET_IP>/<PORT>
```

- **fping** → `fping -asgq` → Prefer for speed and scriptable list generation.
- **ping** → Standard CLI → Prefer for single-host reachability checks.

**Gotchas**: **ICMP Filtering**. Firewall rules may drop ICMP packets, causing active hosts to appear down.

## Service and OS Fingerprinting

Targeted host list available; requires identifying **Domain Controllers**, web servers, and legacy OS versions.

Aggressive service and version detection with script scanning.

```
sudo nmap -v -A -iL <FILE_PATH> -oA <FILE_PATH>
```

- **AD-Specific Protocols**: Prioritize DNS (53), Kerberos (88), LDAP (389, 636), and SMB (445).

**Dangerous / misconfigured settings**:

- Nmap scripted scans can trigger **active vulnerability checks**.
- Scanning industrial equipment (sensors/logic controllers) can cause **system instability** or downtime.

**Gotchas**: **Legacy Systems**. Finding Windows Server 2008 R2 or older often indicates high risk of **EternalBlue** or **MS08-067**, but exploitation requires written client approval due to stability risks.

## Internal User Enumeration

Domain Controller IP identified; requires a valid username list for password spraying without valid credentials.

Harvest valid usernames via Kerberos pre-authentication.

```
kerbrute userenum -d <DOMAIN> --dc <DC_IP> <FILE_PATH> -o <FILE_PATH>
```

> ⚠️ Gap: Kerbrute lacks built-in rate limiting; high-speed enumeration against a small list of accounts or with certain domain policies will trigger **account lockouts** before the sweep completes.

**Gotchas**: **Account Lockout**. Every failed Kerberos pre-authentication attempt counts as a failed login and **will lock out accounts** if thresholds are met.