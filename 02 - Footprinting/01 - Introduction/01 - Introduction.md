# Enumeration Principles and Methodology

### Enumeration Principles

**Enumeration** is a repetitive **loop** of gathering information using both active (scans) and passive (third-party providers) methods. It differs from **OSINT**, which relies exclusively on passive data collection and is considered an independent procedure.

|Principle|Description|
|:--|:--|
|**Perspective**|There is more than meets the eye; consider all points of view.|
|**Visibility**|Distinguish between what is visible and what is hidden.|
|**Understanding**|There are always ways to gain more information; prioritize understanding the target over forcing entry.|

#### Operational Considerations

- **Technical Understanding vs. Exploitation:** The core task of a penetration tester is not to exploit machines, but to **find how they can be exploited**.
- **Avoid Premature Brute-Forcing:** Attempting to brute-force authentication services (SSH, RDP, WinRM) without understanding defensive measures is a noisy approach that often results in **blacklisting** and prevents further testing.
- **Information Depth:** Short-term assessments cannot guarantee the absence of vulnerabilities; long-term analysis by internal teams usually yields a deeper understanding of structure than a multi-week external test.

---

### The 6-Layer Enumeration Methodology

This methodology provides a standardized, systematic approach to explore targets across three levels: **Infrastructure-based**, **Host-based**, and **OS-based** enumeration.

|Layer|Information Categories|Goal|
|:--|:--|:--|
|**1. Internet Presence**|Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances|Identify all possible target systems and externally accessible infrastructure.|
|**2. Gateway**|Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Segmentation, VPN, Cloudflare|Identify security measures protecting the infrastructure and understand the network layout.|
|**3. Accessible Services**|Service Type, Functionality, Configuration, Port, Version, Interface|Understand the purpose of specific services to effectively communicate with and exploit them.|
|**4. Processes**|PID, Processed Data, Tasks, Source, Destination|Identify internal processes and the dependencies between data sources and targets.|
|**5. Privileges**|Groups, Users, Permissions, Restrictions, Environment|Identify internal permissions and find administrative oversights in user/group assignments.|
|**6. OS Setup**|OS Type, Patch Level, Network Config, Config Files, Sensitive Private Files|Collect internal system information to assess the administrative team's security maturity.|

#### Methodology Workflow

1. **Identify Targets:** Focus on finding investigate-able targets within the defined scope (Layer 1).
2. **Analyze Security Controls:** Determine how reachable targets are protected (Layer 2).
3. **Inspect Services:** Examine each destination for its specific functionality and purpose (Layer 3).
4. **Map Data Flow:** Understand the tasks and dependencies of active processes (Layer 4).
5. **Evaluate Access:** Audit the permissions associated with the services and users (Layer 5).
6. **Review System Health:** Gain a complete overview of the internal security and patch levels (Layer 6).

**Note on Internal Environments:** The first two layers (Internet Presence and Gateway) generally do not apply to internal Active Directory infrastructures.

---

### Methodology vs. Cheat Sheet

- **Methodology:** A summary of systematic procedures used to obtain knowledge. It is a **dynamic process** that adapts to the environment.
- **Cheat Sheet:** A collection of specific tools and commands used to execute the methodology. These vary based on the tester's preference and the specific target.

_Note: The provided source material describes the methodology framework but does not list specific tool commands or syntax._