# LLMNR/NBT-NS Poisoning (Windows-based)

### Overview

**Inveigh** is a tool written in PowerShell and C# designed to perform LLMNR and NBT-NS poisoning from a Windows host. It functions similarly to Responder but is utilized when the **attack machine is Windows**, the client provides a Windows testing box, or the operator has gained **local admin** on a Windows host and seeks to escalate privileges or move laterally.

**Attack Implications:** Successful poisoning allows an attacker to capture NTLMv1/v2 hashes and cleartext credentials. These can be cracked offline or used for further enumeration (e.g., via BloodHound) to identify high-value targets like Domain Admins.

---

### Supported Protocols

Inveigh can listen to and spoof several protocols:

- **IPv4/IPv6:** LLMNR, DNS, mDNS, NBNS.
- **Application Layer:** HTTP, HTTPS, SMB, LDAP, WebDAV, and Proxy Auth.
- **Others:** DHCPv6, ICMPv6.

---

### Operational Workflows

#### Technique 1: PowerShell Inveigh

The PowerShell version is the original implementation. While no longer actively updated, it remains useful for quick execution if the environment allows PowerShell scripts.

1. **Import the module:**
    
    ```
    Import-Module .\Inveigh.ps1
    ```
    
2. **Inspect available parameters:**
    
    ```
    (Get-Command Invoke-Inveigh).Parameters
    ```
    
3. **Execute poisoning with console and file output:**
    
    ```
    Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
    ```
    
    - **Scenario Context:** Use these flags to ensure LLMNR/NBNS spoofing is active and that results are both visible and logged for later retrieval.

#### Technique 2: C# Inveigh (InveighZero)

This is the actively maintained version that combines the original C# proof-of-concept with ported PowerShell code.

1. **Execute the tool with defaults:**
    
    ```
    .\Inveigh.exe
    ```
    
2. **Enter the interactive console:** Press **ESC** while the tool is running to enter or exit the interactive mode.
3. **Retrieve captured data:**
    - **Unique Hashes:** Use `GET NTLMV2UNIQUE` to see one hash per user.
    - **Username List:** Use `GET NTLMV2USERNAMES` to identify accounts for further enumeration.
4. **Stop the attack:** Use `STOP` within the interactive console.

---

### Command Reference

#### Inveigh Interactive Console Commands

|Command|Description|
|:--|:--|
|`GET NTLMV2UNIQUE`|Retrieves one captured NTLMv2 hash per user.|
|`GET NTLMV2USERNAMES`|Lists captured usernames and source IPs/hostnames.|
|`GET CLEARTEXT`|Displays captured cleartext credentials.|
|`GET LOG`|Displays log entries (can filter with a search string).|
|`RESUME`|Resumes real-time console output.|
|`STOP`|Terminates the Inveigh session.|

#### Key Parameters (PowerShell Version)

|Parameter|Description|
|:--|:--|
|`ADIDNSHostsIgnore`|List of hosts to ignore for ADIDNS poisoning.|
|`LLMNRTTL`|Sets the Time to Live for LLMNR responses.|
|`HTTPPort` / `HTTPSPort`|Specifies the ports for the web listeners.|

---

### Failures and Edge Cases

- **Socket Errors:** A "forbidden by access permissions" exception or "Failed to start HTTP listener" warning usually indicates **port 80 is already in use** on the host, preventing Inveigh from starting its own listener on that port.
- **Disabled Spoofers:** The tool output will explicitly flag disabled spoofers with a `[ ]` or `[disabled]` tag. For example, mDNS is often disabled by default and will not send responses unless configured.

---

### Remediation and Detection

#### Remediation Measures

|Target|Mitigation Action|Implementation|
|:--|:--|:--|
|**LLMNR**|Disable via GPO|**Computer Configuration** > **Admin Templates** > **Network** > **DNS Client** > Enable **"Turn OFF Multicast Name Resolution"**.|
|**NBT-NS**|Disable locally or via Script|Use a **PowerShell startup script** pushed via GPO to modify the registry key `HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces`.|
|**NTLM Relay**|Enable SMB Signing|Prevents attackers from relaying captured hashes to other hosts.|

#### PowerShell Script to Disable NBT-NS

Apply this via **GPO Startup Scripts** to automate across the domain:

```
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey | foreach {
    Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose
}
```

#### Detection Strategies

- **Honeytokens:** Inject LLMNR/NBT-NS requests for non-existent hosts; any response indicates spoofing activity.
- **Traffic Monitoring:** Monitor for unexpected traffic on **UDP 5355** (LLMNR) and **UDP 137** (NetBIOS).
- **Registry Monitoring:** Watch `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` for changes to the `EnableMulticast` DWORD (0 = Disabled).
- **Event Logs:** Monitor for **Event IDs 4697 and 7045**.