## METHODOLOGY

1. Differentiate **OSINT** from **Enumeration**. OSINT is strictly **passive** third-party gathering; Enumeration requires **active** target interaction.
2. Initiate the enumeration loop. Use discovered data to feed subsequent scans recursively.
3. Execute **Infrastructure-based enumeration**. Define the external scope and identify defensive boundaries.
4. Execute **Host-based enumeration**. Map specific services and their internal logic.
5. Execute **OS-based enumeration**. Audit internal configurations and permission sets once a foothold is established.
6. Map the "labyrinth" of gaps. Identify multiple vulnerabilities but prioritize those that provide a direct path through the layers.

---

## Systematic Enumeration Principles

When to use: During any phase where progress stalls or before initial interaction to avoid **blacklisting**.

- Prioritize technical understanding over exploitation. Finding how a system _can_ be exploited is the core task, not the exploit itself.
- Analyze visible and non-visible components. Experience identifies infrastructure elements that are not immediately apparent.
- Avoid immediate **brute-forcing** on authentication services (SSH, RDP, WinRM). This is a **noisy method** that triggers defensive measures and results in **blacklisting**.

> ⚠️ Gap: Standard enumeration techniques for Layer 1 and 2 (Internet Presence/Gateway) **do not apply** to internal **Active Directory** or intranet environments.

## Layer 1: Internet Presence

When to use: Starting an external black box engagement with a defined or undefined target list.

- Identify targets: **Domains**, **Subdomains**, **vHosts**, **ASN**, **Netblocks**, and **IP Addresses**.
- Identify infrastructure: Locate **Cloud Instances** and existing **Security Measures**.

## Layer 2: Gateway Identification

When to use: After defining targets to determine how they are protected and positioned in the network.

- Identify security obstacles: **Firewalls**, **DMZ**, **IPS/IDS**, **EDR**, **Proxies**, and **NAC**.
- Identify entry points: **VPN**, **Cloudflare** implementations, and **Network Segmentation**.

## Layer 3: Accessible Services

When to use: Assessing destination IPs to understand their purpose and communication requirements.

- Examine interfaces: Determine **Service Type**, **Port**, **Version**, and specific **Functionality**.
- Audit setup: Analyze the service **Configuration** and available **Interface** options.

## Layer 4: Process Analysis

When to use: After service identification to track how the system handles user or system-generated data.

- Trace execution: Identify **PID** and the **Tasks** associated with active services.
- Map dependencies: Identify the **Source**, **Destination**, and **Processed Data** for every command.

## Layer 5: Privilege Identification

When to use: Evaluating access levels within services or Active Directory environments.

- Identify permissions: Audit **Groups**, **Users**, and specific **Permissions** or **Restrictions**.
- Exploit administrative oversight: Look for users with **multiple administration areas** or overlooked **environment** privileges.

## Layer 6: OS Setup and Environment

When to use: Once internal access is achieved to evaluate administrative security standards.

- Identify system components: Determine **OS Type**, **Patch Level**, and **Network config**.
- Locate sensitive data: Search for **Configuration files**, **OS Environment** variables, and **sensitive private files**.

### Gotchas

- **Brute-forcing** early in the engagement often leads to **blacklisting** and makes further testing impossible.
- Failing to understand the target's functionality leads to **waste of time** and **damage** to the infrastructure.
- **Not all gaps** or vulnerabilities found in the "labyrinth" lead to internal access; time-limited tests require identifying the _correct_ gap.