# Initial Domain Enumeration Methodology

Active Directory enumeration should be approached in **progressive stages** to avoid missing data or being overwhelmed by the volume of information. The standard workflow moves from **passive identification** to **active validation** and finally **targeted probing**.

### Key Data Points for Enumeration
| Data Point | Description |
| :--- | :--- |
| **AD Users** | Valid accounts for future password spraying. |
| **AD Joined Computers** | Domain Controllers, file servers, SQL, Exchange, and web servers. |
| **Key Services** | Kerberos, NetBIOS, LDAP, and DNS. |
| **Vulnerable Hosts** | Legacy systems or misconfigured services for initial footholds. |

---

## Phase 1: Passive Identification
Use these techniques to "listen" to the network without sending packets. This is critical for **black box** or **evasive** assessments where discovery must be done blindly.

### Network Traffic Analysis
**Goal:** Identify active hosts and traffic patterns (ARP, MDNS, LLMNR) via broadcast domains.

1. **Start a packet capture** on the local interface to observe layer two packets and MDNS queries.
2. **Review PCAP files** for hostnames and IP addresses to build an initial target list.

| Tool | Command | Purpose |
| :--- | :--- | :--- |
| **Wireshark** | `sudo -E wireshark` | GUI-based traffic analysis. |
| **tcpdump** | `sudo tcpdump -i <INTERFACE>` | CLI-based traffic capture for non-GUI hosts. |

### Passive Service Analysis
**Goal:** Utilize **Responder** in **Analyze Mode** to passively map the domain without sending poisoned packets.

| Command | Parameter | Description |
| :--- | :--- | :--- |
| `sudo responder -I <INTERFACE> -A` | `-A` | Enables **Analyze Mode** to listen for LLMNR, NBT-NS, and MDNS requests. |

---

## Phase 2: Active Host Discovery
Once passive data is collected, use active probes to validate which hosts are live on the subnet.

### ICMP Sweeping
**Goal:** Rapidly identify live hosts in a CIDR range using a **round-robin** approach to minimize wait times.

| Command                       | Description                                                     |
| :---------------------------- | :-------------------------------------------------------------- |
| `fping -asgq <TARGET_SUBNET>` | Performs a quiet ICMP sweep and outputs only live IP addresses. |

**Fping Parameters:**
*   `-a`: Show only targets that are alive.
*   `-s`: Print final stats.
*   `-g`: Generate target list from CIDR.
*   `-q`: Hide per-target results (quiet mode).

---

## Phase 3: Service and Vulnerability Scanning
**Goal:** Identify critical AD infrastructure (Domain Controllers) and potential "quick wins" like legacy operating systems.

1. **Create a host list** from fping and Responder results.
2. **Perform an aggressive scan** to determine services, versions, and OS details.

| Command | Purpose |
| :--- | :--- |
| `sudo nmap -v -A -iL hosts.txt -oA <OUTPUT_NAME>` | Aggressive scan of identified hosts to find LDAP, Kerberos, and SMB. |

### High-Value Targets & Risk Factors
| Feature | Attack Implication |
| :--- | :--- |
| **Legacy OS (e.g., Server 2008 R2)** | High potential for **EternalBlue** or **MS08-067**; provides an easy SYSTEM shell. |
| **Port 88 (Kerberos)** | Identifies the host as a **Domain Controller**. |
| **Port 389/636 (LDAP)** | Used for domain enumeration and identifying the naming standard. |

> [!WARNING]
> **Operational Risk:** Active scans and Nmap scripts can disrupt sensitive equipment like **sensors or logic controllers**. Always obtain written approval before exploiting legacy systems, as they may be unstable.

---

## Phase 4: Username Enumeration
**Goal:** Establish a valid list of domain users for password spraying when no credentials are provided.

### Kerbrute Enumeration
**Kerbrute** is preferred because **Kerberos pre-authentication failures** often do not trigger standard security logs, making it stealthier than other methods.

**Setup (Compilation):**
1. Clone the repository.
2. Run `make linux` or `make all` to generate binaries.
3. Move the binary to the local path: `sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute`.

**Execution:**
| Command | Purpose |
| :--- | :--- |
| `kerbrute userenum -d <DOMAIN> --dc <TARGET_IP> <USER_LIST> -o <OUTPUT_FILE>` | Brute-forces valid AD accounts via Kerberos Pre-Authentication. |

> [!CAUTION]
> **Account Lockout:** While stealthier, failed pre-authentication counts as a failed login and **WILL lock out accounts** if thresholds are exceeded.

---

## Attack Implications: SYSTEM Access
Gaining **NT AUTHORITY\SYSTEM** on a domain-joined host is nearly equivalent to having a domain user account. It allows an attacker to:
*   Enumerate Active Directory by **impersonating the computer account**.
*   Open opportunities for cleartext credential harvesting or NTLM hash retrieval.