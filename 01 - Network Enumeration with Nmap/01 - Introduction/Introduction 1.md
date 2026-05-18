## System Enumeration Methodology

1. Scan for available hosts and open ports using raw packets to identify the attack surface.
2. Determine service names, versions, and operating systems to narrow down potential vulnerabilities.
3. Manually interact with discovered services to uncover information that automated tools miss due to protocol limitations or security measures.
4. Analyze service responses for misconfigurations or signs of security neglect, such as over-reliance on firewalls or GPOs.

---

## Network Scanning

### TCP SYN Scanning

Host is up and requires high-speed port identification without establishing full sessions.

Standard stealth scan to probe thousands of ports per second

```
sudo nmap -sS <TARGET_IP>
```

Identify ports and service states to drive subsequent enumeration

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1
```

- **Dangerous / misconfigured settings**
    - Administrators relying exclusively on firewalls or GPOs while neglecting individual service security.
- **Edge cases**
    - Automated tools with set timeouts may mark a port as closed, filtered, or unknown if the service is slow to respond.
- **Gotchas**
    - **Port marked as closed** by Nmap can result from aggressive timing, potentially hiding a viable entry point.

> ⚠️ Gap: Nmap requires root/sudo privileges to craft the raw packets needed for SYN scans, otherwise it may default to a full TCP connect scan which is noisier.

## Service Enumeration

### Manual Service Interaction

Automated scans are complete but have failed to identify clear vulnerabilities or have timed out.

General Nmap syntax for version and OS detection

```
nmap <SCAN_TYPES> <OPTIONS> <TARGET_IP>
```

- **Gotchas**
    - **Tool-only reliance** fails when security measures block automated probes; manual interaction is required to bypass these restrictions.
    - **Handshake never completes** during SYN scans, which may prevent certain application-level data from being gathered compared to manual interaction.