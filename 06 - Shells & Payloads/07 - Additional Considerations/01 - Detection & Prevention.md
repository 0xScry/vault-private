# Detection & Prevention Methodology

## MITRE ATT&CK Framework

The **MITRE ATT&CK Framework** provides a knowledge base of adversary tactics and techniques based on real-world observations. Understanding these allows for better identification of active shells and payload execution.

### Notable Tactics and Techniques

|Tactic|Description|Attack Implications|
|:--|:--|:--|
|**Initial Access**|Compromising public-facing hosts (Web Apps), misconfigured services (SMB), or authentication protocol bugs.|Provides a foothold in the network; often occurs on a **bastion host**.|
|**Execution**|Running attacker-supplied code via PowerShell one-liners, Metasploit exploits, or remote file calls.|Facilitates shell access and interactive control over the target.|
|**Command & Control (C2)**|Establishing interactive access using standard ports/protocols (HTTP/S, DNS, NTP) or allowed apps (Slack, Discord).|Allows for follow-on actions and data exfiltration within the victim network.|

## Network Visibility and Detection

Payloads must communicate over the network to be effective. **Network visibility** is essential to identify these communications.

### Detection Techniques

1. **Baseline Traffic:** Utilize cloud-based network controllers to establish a baseline of traffic usage, protocols, and applications. Any **deviation from the norm** becomes highly visible.
2. **Deep Packet Inspection (DPI):** Use network security appliances to act as a network-level anti-virus. DPI can detect and block payloads even if they execute successfully on a host.
3. **NetFlow Analysis:** Monitor flow data to identify suspicious connections, such as frequent communication on non-standard or known malicious ports (e.g., `<PORT>`).
4. **Protocol Inspection:** Identify unencrypted traffic (e.g., **Netcat**). Unencrypted streams allow defenders to capture and read every command sent between the `<ATTACK_IP>` and `<TARGET_IP>`.

### Persistence Detection via Command Monitoring

When NetFlow data is paired with **command-line logging**, defenders can identify malicious actions like unauthorized user creation.

|Goal|Command|
|:--|:--|
|Create a new user for persistence|`net user <USERNAME> <PASSWORD> /add`|
|Elevate user privileges|`net localgroup administrators <USERNAME> /add`|

## End Device Protection

**End devices** (workstations, servers, etc.) are primary targets because they provide the CLI interfaces used for administration and automation.

### Windows Hardening Measures

- **Windows Defender:** Must remain enabled to prevent payload execution and shell establishment.
- **Defender Firewall:** Keep all profiles (Domain, Private, and Public) active. Only allow exceptions for approved applications via change management.
- **Patch Management:** Ensure all hosts receive updates immediately after release to close known vulnerabilities.
- **Logging:** Implement **command-line logging** to triage events and determine if an incident has occurred.

## Mitigation Reference

### Dangerous Misconfigurations

|Misconfiguration|Risk|
|:--|:--|
|**Unencrypted Protocols**|Allows full visibility of attacker commands via packet capture.|
|**Disabled AV/Firewall**|Removes the primary barrier to payload execution and callback establishment.|
|**Lack of Logging**|Prevents defenders from distinguishing between admin activity and malicious shells.|

### Security Controls

|Control|Function|
|:--|:--|
|**Network Segmentation**|Limits the "blast radius" and prevents lateral movement.|
|**IDS/IPS**|Detects and prevents known malicious signatures and behaviors at the network level.|
|**Endpoint Protection (EDR/AV)**|Identifies and blocks malicious files and scripts on the host.|
|**Deep Packet Inspection**|Inspects the contents of packets to identify hidden or obfuscated threats.|