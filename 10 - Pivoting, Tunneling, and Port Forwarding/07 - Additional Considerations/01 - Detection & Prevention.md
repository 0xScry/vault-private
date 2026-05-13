1. **Perimeter and Infrastructure Assessment**
    
    - Identify if environment is hybrid-cloud (AWS, Azure, GCP) or strictly on-premises.
    - Map critical assets and network diagrams to identify segmentation gaps.
    - Check for BYOD presence; personal devices often lack enterprise-grade hardening and provide initial pivot points.
2. **Network Segmentation and Access Control Audit**
    
    - Verify if standard users (e.g., HR) can reach infrastructure management panels or routers.
    - Determine if Out of Band (OOB) networks are used for management; if not, infrastructure devices are accessible from general user segments.
3. **Protocol and Port Selection**
    
    - Scan for non-standard port usage (e.g., HTTP/HTTPS on port 444) to bypass basic port filtering.
    - Evaluate DNS resolution paths; if internal hosts can resolve externally, DNS tunneling is viable.
4. **Tunneling and Proxy Execution**
    
    - If SSH/RDP is available, check for MFA enforcement; compromise of credentials alone fails if MFA is active.
    - Implement protocol tunneling (SSH or DNS) to bypass firewalls and mask traffic.
    - Use proxy points to distribute traffic and avoid direct connection between the victim and attack infrastructure.

---

## Protocol Tunneling

**When to use** Need to bypass firewalls or hide communications within allowed traffic like SSH or DNS.

**Commands**

> ⚠️ Gap: The source describes the logic of tunneling traffic through SSH and DNS but does not provide the specific command-line syntax for execution.

**Dangerous / misconfigured settings**

- Internal hosts allowed to perform external DNS resolution directly instead of through a designated internal DC/DNS server.
- Lack of monitoring for beaconing patterns or request timing in encrypted channels.

**Gotchas** **Beaconing detection** can trigger alerts even in encrypted tunnels if requests follow a consistent temporal pattern.

## Remote Service Access

**When to use** Targeting internal movement via SSH or RDP after initial credential access.

**Commands**

> ⚠️ Gap: The source identifies the use of Remote Services (T1021) and External Remote Services (T1133) but lacks the specific commands for initiating these connections.

**Dangerous / misconfigured settings**

- Management services (SSH/RDP) exposed to general user segments instead of restricted OOB networks.
- Remote access permissions granted to broad user groups rather than restricted, duty-separated accounts.

**Gotchas** **MFA enforcement** will block access even with valid passwords or hashes.

## Proxying and Traffic Distribution

**When to use** Requirement to mask the origin of attack infrastructure or prevent direct victim-to-attacker netflow logs.

**Commands**

> ⚠️ Gap: The source identifies proxy use (T1090) for infrastructure masking but provides no configuration syntax for proxy tools.

**Tool comparison**

- Diagrams.net
    - `diagrams.net`
    - Prefer when a free, visual network documentation tool is required for baselining.
- Netbrain
    - [No syntax provided]
    - Prefer for interactive access and automated mapping of all appliances in a diagram.

**Edge cases**

- Environments with strictly maintained allow-lists for domains/IPs make proxying difficult as all non-explicitly allowed traffic is blocked.

**Gotchas** **Netflow baselining** by defenders can identify abnormal traffic patterns if the environment's "normal" state is well-documented.

## Non-Standard Port Communication

**When to use** Standard ports (80, 443) are heavily monitored or restricted, requiring traffic to be moved to atypical ports.

**Commands** Example of using a non-standard port for HTTPS communication

```
<TARGET_IP>:444
```

**Gotchas** **NIPS/NIDS systems** and protocol/port pairing analysis can identify suspicious traffic (e.g., HTTPS over 444) if a solid baseline exists.