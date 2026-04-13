# Detection & Prevention

As a penetration tester, understanding defensive mitigations is critical for providing **remediation advice**. Defensive strategies are categorized into **People, Process, and Technology** to address hardware, software, and human vulnerabilities.

### Establishing a Network Baseline

Defenders must identify and investigate new hosts, unauthorized applications, and unique traffic patterns. An audit of these elements should occur **annually or every few months**.

|Category|Items to Document & Track|
|:--|:--|
|**Inventory**|All hosts, servers, workstations, and mobile devices.|
|**Applications**|Application catalog vs. unauthorized software/tools.|
|**Network**|Visual network diagrams (e.g., **Netbrain**, **diagrams.net**), critical assets, and net flow.|

### Defense-in-Depth Methodology

#### 1. The Human Element (People)

Users are often the weakest link; securing them prevents "easy wins".

- **MFA Implementation:** Use two or more factors (e.g., something you have, know, or are) especially for **administrative accounts**.
- **BYOD Risks:** Personally owned devices (laptops/smartphones) often lack organizational security administration, allowing malware to bridge from home environments to the **employee network**.

#### 2. Administrative Control (Process)

Defined policies ensure accountability and structured responses.

- **Security Operations:** Utilize a **SOC** (internal or as-a-service) for 24/7 monitoring.
- **Incident Response:** Maintain a practiced **disaster recovery plan** and IR plan to handle breaches.

#### 3. Infrastructure Hardening (Technology)

Visibility and segmentation are the primary deterrents to lateral movement.

- **Visibility:** Implement a **SIEM** to correlate host and infrastructure logs.
- **Segmentation:** Ensure standard users (e.g., HR) cannot access network infrastructure like **switches, routers, or internal admin panels**.
- **Hardening:** Periodically check for **legacy misconfigurations** and prioritize patching based on the **CIA triad**.

---

### MITRE ATT&CK Mitigation Reference

Use this table to map offensive actions discovered during testing to specific defensive controls.

|TTP|MITRE Tag|Mitigation / Detection Strategy|
|:--|:--|:--|
|**External Remote Services**|T1133|Use **firewalls** to segment the environment; block internal protocols from reaching the internet; require **VPN** for service access.|
|**Remote Services**|T1021|Enforce **MFA** for SSH/RDP; limit remote access permissions; use host-based firewalls to restrict connections to **authorized networks** only.|
|**Management Traffic**|N/A|Expose infrastructure management (routers/switches) only to an **Out Of Band (OOB) network** to prevent pivoting from user segments.|
|**Non-Standard Ports**|T1571|Compare traffic against a known **port/protocol baseline**; use **NIPS/NIDS** to identify protocol mismatches (e.g., HTTP over port 444).|
|**Protocol Tunneling**|T1572|Disallow external DNS resolution except for designated **DNS servers**; monitor for **Beaconing patterns** indicating C2 channels.|
|**Proxy Use**|T1090|Maintain **allow/block lists** for domains and IP addresses; require explicit approval for outbound net flow.|
|**Living off the Land (LOTL)**|N/A|Establish a **behavioral baseline** for command shells; deploy **EDR/AV** solutions; feed all logs into a **SIEM** for early-stage detection.|

---

### Operational Workflow: Outside-In Assessment

When assessing a client's posture, follow this sequence to mirror real-world threats:

1. **Perimeter Evaluation:** Identify infrastructure on-premises and in the **hybrid-cloud** (AWS, Azure, GCP).
2. **External Access:** Verify if external services are gated by **firewalls and VPNs**.
3. **Internal Movement:** Test if a compromised user can reach **Domain Controllers** or **File Shares** due to lack of segmentation.
4. **Detection Test:** Determine if tools, unauthorized traffic, or **protocol tunnels** trigger alerts in the **SIEM/EDR**.