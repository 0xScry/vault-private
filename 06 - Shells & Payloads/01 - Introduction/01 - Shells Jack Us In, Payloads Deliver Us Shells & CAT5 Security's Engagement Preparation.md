# Shells & Payloads Fundamentals

### Shell Overview

A **shell** is a program providing an interface to input instructions and view text output (e.g., **Bash, Zsh, cmd, PowerShell**). In a security context, a shell is the result of exploiting a vulnerability or bypassing security to gain **interactive access** to a host.

### Operational Value: Why Establish a Shell

Establishing a shell is a primary goal following enumeration and exploit identification because it provides **remote control** over the target operating system.

|Benefit|Impact on Engagement|
|:--|:--|
|**System Access**|Grants direct access to the OS, system commands, and the file system.|
|**Post-Exploitation**|Unlocks vectors for **privilege escalation**, **pivoting**, and **file transfers**.|
|**Persistence**|Allows the tester to maintain access over time to complete objectives.|
|**Efficiency**|Enables the use of attack tools, data exfiltration, and automated actions.|
|**Stealth**|CLI shells are harder to detect than graphical shells (VNC/RDP) and faster to navigate.|

### Shell Perspectives & Environments

Techniques for gaining and using shells vary based on the environment and the entry vector.

|Perspective|Description|
|:--|:--|
|**Computing**|The text-based userland environment used to submit instructions (e.g., Bash, PowerShell).|
|**Exploitation**|The result of triggering a vulnerability to gain interactive access (e.g., using **EternalBlue** to gain a `cmd` prompt).|
|**Web Shell**|A script uploaded via a web vulnerability (like file upload) that allows an attacker to issue instructions and access files through a **browser window**.|

### Operational Workflow: Gaining Access

1. **Enumeration:** Identify promising exploits on the target.
2. **Exploitation:** Trigger a vulnerability to bypass security measures.
3. **Payload Delivery:** Use a **payload** to establish the remote shell session.
4. **Interactive Control:** Gain access to the CLI to begin post-exploitation tasks.

### Attack Implications

- **Access Level:** Without a shell, a tester is strictly limited in their ability to interact with the target.
- **Detection:** Command-line interfaces are preferred over graphical interfaces to avoid detection and increase operational speed.
- **Automation:** CLI access allows for the automation of complex attack chains.