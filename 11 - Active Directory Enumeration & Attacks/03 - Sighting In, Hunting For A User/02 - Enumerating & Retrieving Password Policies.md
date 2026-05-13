## Remote Credentialed Enumeration (Linux)

You have valid domain credentials and need a readable summary of the domain password policy via SMB.

CrackMapExec for a formatted summary of lockout thresholds and complexity requirements:

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --pass-pol
```

- **Tool comparison**
    - CrackMapExec -> `crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --pass-pol` -> Prefer for quick, readable summaries and automated credential testing.
    - rpcclient -> `rpcclient -U <USERNAME> <TARGET_IP>` -> Prefer for manual granular queries if CME is flagged.

**Gotchas**: **Invalid credentials** will prevent policy retrieval; verify account status before spraying.

## SMB Null Session Enumeration (Linux)

Unauthenticated on the network and targeting legacy Domain Controllers or upgraded systems where **anonymous access** remains enabled,.

1. Verify NULL session access:

```
rpcclient -U "" -N <TARGET_IP>
```

2. Confirm domain info once connected:

```
querydominfo
```

3. Retrieve password policy via RPC:

```
getdompwinfo
```

Legacy enum4linux for quick unauthenticated SMB checks:

```
enum4linux -P <TARGET_IP>
```

Modern rewrite for JSON/YAML export to feed other tools:

```
enum4linux-ng -P <TARGET_IP> -oA <FILE_PATH>
```

- **Tool comparison**
    
    - rpcclient -> `getdompwinfo` -> Prefer for manual verification of specific RPC calls.
    - enum4linux-ng -> `enum4linux-ng -P <TARGET_IP> -oA <FILE_PATH>` -> Prefer for engagements requiring structured data export for reporting or further automation,.
- **Dangerous / misconfigured settings**
    
    - **Legacy DC upgrades** that carry forward insecure default configurations from older Windows Server versions.
    - **Anonymous access** granted to shares in earlier Windows environments.

**Gotchas**: **Random user sessions** often fail (STATUS_LOGON_FAILURE) even when NULL sessions are permitted.

## SMB Null Session Validation (Windows)

Positioned on a Windows host without specialized tools and need to confirm if an anonymous IPC$ connection is possible.

Establish a NULL session to the IPC share:

```
net use \\<TARGET_IP>\ipc$ "" /u: ""
```

**Gotchas**: **Account disabled** (Error 1331) or **Account locked out** (Error 1909) will block the session even if the NULL session configuration is present.

## LDAP Anonymous Bind Enumeration (Linux)

SMB is restricted but the target allows unauthenticated LDAP queries, common in legacy Windows Server 2003 or misconfigured application service accounts.

Retrieve password policy fields including history length and lockout duration:

```
ldapsearch -H ldap://<TARGET_IP> -x -b "DC=<DOMAIN>,DC=<LOCAL>" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

- **Edge cases**
    - Newer versions of `ldapsearch` require `-H` for the URI; `-h` is deprecated.

**Gotchas**: **Windows Server 2003 and later** default to requiring authentication; anonymous binds only work if explicitly misconfigured by an admin.

## Local Credentialed Enumeration (Windows)

You have an active shell on a domain-joined Windows host and need to enumerate the policy using built-in binaries or common post-exploitation scripts,.

Built-in command when tool transfer (PowerView/CME) is restricted by AV/EDR:

```
net accounts
```

PowerView to retrieve full policy details including the specific GPO name and Kerberos settings:

```
Get-DomainPolicy
```

- **Tool comparison**
    - net.exe -> `net accounts` -> Use for minimal footprint; provides core lockout and length data.
    - PowerView -> `Get-DomainPolicy` -> Use for comprehensive details, including `LockoutBadCount` and `GPODisplayName`,.

**Gotchas**: **Execution policy** or **AV/EDR** may block PowerShell scripts; revert to `net.exe` for basic policy data.

## Password Spray Strategy

The retrieved policy shows a **Lockout threshold** and **Lockout duration**, requiring a calculated spray interval to avoid detection or account lockouts.

- **Dangerous / misconfigured settings**
    
    - **Lockout threshold of 3** bad attempts requires extreme caution; one wrong guess can trigger a lockout.
    - **Manual admin intervention** required for unlocking accounts.
- **Edge cases**
    
    - If the policy cannot be retrieved, limit to 1 spray attempt and wait over 1 hour between cycles.

**Gotchas**: **Locking out all accounts** in an organization is a critical engagement failure; always prioritize the lockout threshold over speed.