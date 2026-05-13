## Methodology

1. Verify environment for local enumeration tools by checking for the **ActiveDirectory module** presence via `Get-Module`.
2. Enumerate domain basics and trusts using native cmdlets to establish the forest layout and identify potential cross-domain attack paths.
3. Identify high-value accounts by filtering for **ServicePrincipalName (SPN)** attributes and analyzing recursive group memberships to find elevation targets.
4. If native tools are insufficient or you need deeper situational awareness, use **PowerView** to check for local admin access and unprivileged modification rights.
5. In environments with **hardened PowerShell usage**, switch to **SharpView** to execute .NET-based enumeration.
6. Automate the discovery of credentials and sensitive configuration files in network shares using **Snaffler**.
7. Execute **SharpHound** to collect relationship data and ingest it into BloodHound to visualize non-obvious attack paths to Domain Admin.

---

## Built-in AD PowerShell Enumeration

Landed on a Windows host with a valid domain context and need to blend in with standard admin activity.

Check for the availability of the ActiveDirectory module before attempting enumeration

```
Get-Module
```

Import the module to enable AD-specific cmdlets if it is not currently loaded

```
Import-Module ActiveDirectory
```

Identify domain SID, functional level, and child domains to scope the environment

```
Get-ADDomain
```

Enumerate accounts for Kerberoasting by filtering for the ServicePrincipalName property

```
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

List all domain trust relationships to identify forest boundaries and transitive trusts

```
Get-ADTrust -Filter *
```

List all groups to find high-value targets like Backup Operators or Domain Admins

```
Get-ADGroup -Filter * | select name
```

Enumerate specific group membership to find users who inherit sensitive permissions

```
Get-ADGroupMember -Identity "<SERVICE_NAME>"
```

- **Tool comparison**
    
    - ActiveDirectory Module
        - `Get-ADGroupMember -Identity "<SERVICE_NAME>"`
        - Prefer when stealth is a priority and you need to blend in with legitimate administrator PowerShell traffic.
- **Dangerous / misconfigured settings**
    
- ServicePrincipalName (SPN) populated on standard user accounts.
    
- Service accounts placed in sensitive groups like **Backup Operators**.
    
- **Gotchas**
    
- **Module not imported** will result in "command not found" errors; always check `Get-Module` first.
    

## PowerView Awareness and SharpView Evasion

Landed on a host and need deep situational awareness, including local admin rights and nested group analysis.

Retrieve detailed attributes for a target user including UAC flags and group membership

```
Get-DomainUser -Identity <USERNAME> -Domain <DOMAIN> | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
```

Perform recursive group membership lookup to find users who inherit Domain Admin rights via nested groups

```
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

Identify all trust relationships and attributes like Forest Transitive or Within Forest in one sweep

```
Get-DomainTrustMapping
```

Verify if the current user has local administrative privileges on a specific remote host

```
Test-AdminAccess -ComputerName <TARGET_IP>
```

Identify all Kerberoastable accounts by searching for the SPN attribute across the domain

```
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

Execute user enumeration via .NET when PowerShell execution is restricted or heavily monitored

```
.\SharpView.exe Get-DomainUser -Identity <USERNAME>
```

- **Tool comparison**
    
    - PowerView
        - `Get-DomainGroupMember -Identity "<GROUP_NAME>" -Recurse`
        - Prefer for manual, granular enumeration and checking for specific misconfigurations like local admin access.
    - SharpView
        - `.\SharpView.exe Get-DomainUser -Identity <USERNAME>`
        - Prefer when **PowerShell usage is hardened** or you need to avoid PowerShell-based logging/detection.
- **Dangerous / misconfigured settings**
    
- **Nested group membership** where non-admin groups inherit high-level privileges.
    
- **Local Admin rights** granted to the "Domain Users" group on sensitive workstations or servers.
    
- UserAccountControl flags like **DONT_REQ_PREAUTH** or **DONT_EXPIRE_PASSWORD**.
    
- **Gotchas**
    
- **PowerView is deprecated** in its original form; ensure you are using a maintained fork for modern AD environments.
    

## Share Pillaging with Snaffler

Landed as a domain user and need to find passwords, SSH keys, or config files across all reachable network shares.

Automate share discovery and file content searching with real-time console output and logging

```
.\Snaffler.exe -d <DOMAIN> -s -v data -o <FILE_PATH>
```

- **Dangerous / misconfigured settings**
    
- Sensitive file extensions like `.kdb`, `.key`, `.ppk`, or `.sqldump` stored in world-readable shares.
    
- **Overly permissive shares** (e.g., IT or Infrastructure shares) accessible by standard domain users.
    
- **Gotchas**
    
- **Snaffler must run in a domain-user context** or from a domain-joined host to authenticate to shares.
    

## Relationship Mapping with SharpHound

Need to visualize complex attack paths and identify non-obvious ways to escalate from an unprivileged user to Domain Admin.

Run all collection methods and compress results into a zip file for exfiltration

```
.\SharpHound.exe -c All --zipfilename <FILE_PATH>
```

Collect data while minimizing traffic to the Domain Controller by focusing on LDAP queries

```
.\SharpHound.exe -c DCOnly --stealth
```

- **Edge cases**
    
- Use `--stealth` and `DCOnly` when you need to avoid high-volume traffic to member servers for session/local admin enumeration.
    
- **Gotchas**
    
- **Non-live legacy records** may appear in the graph; always validate if a host is "live" before attempting an exploit.
    
- **Fragile legacy systems** (e.g., Windows 7/2008) identified in BloodHound should be handled with caution as they may crash under heavy load.