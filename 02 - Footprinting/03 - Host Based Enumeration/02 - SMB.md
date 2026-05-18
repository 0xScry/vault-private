## SMB Service Discovery

Ports 139 and 445 are open on a target.

Service discovery and default script scanning

```
sudo nmap <TARGET_IP> -sV -sC -p139,445
```

- **Slow execution** — Nmap scripts can take significant time; manual interaction usually provides deeper service details.

## SMB Share Enumeration

Identifying accessible directories and permissions when null sessions or credentials are available.

List shares anonymously using a null session

```
smbclient -N -L //<TARGET_IP>
```

Standard share enumeration for quick permission overview

```
smbmap -H <TARGET_IP>
```

Alternative share enumeration with NULL credentials

```
crackmapexec smb <TARGET_IP> --shares -u '' -p ''
```

Automated comprehensive enumeration

```
./enum4linux-ng.py <TARGET_IP> -A
```

### Tool comparison

- `smbclient` -> `smbclient -N -L //<TARGET_IP>` -> prefer for standard manual verification.
- `smbmap` -> `smbmap -H <TARGET_IP>` -> prefer for a clean visual of permissions across all shares.
- `crackmapexec` -> `crackmapexec smb <TARGET_IP> --shares -u '' -p ''` -> prefer for identifying readable/writable shares across multiple hosts.
- `enum4linux-ng` -> `./enum4linux-ng.py <TARGET_IP> -A` -> prefer for broad, automated data collection including OS and policy info.

### Dangerous / misconfigured settings

- `browseable = yes` — Allows listing the share even if the user lacks specific access.
    
- `guest ok = yes` — Allows connection without a password.
    
- `map to guest = bad user` — Maps failed login attempts to a guest account.
    
- **Tool inconsistency** — Different automated tools may return different results based on their internal programming; always verify findings manually.
    

## File Interaction via SMB

Accessing, downloading, or inspecting files within an identified share.

Connect to a specific share

```
smbclient //<TARGET_IP>/<SHARE_NAME>
```

Download a specific file to the local machine

```
get <FILE_PATH>
```

Execute a local system command without dropping the SMB session

```
!<cmd>
```

- `read only = no` or `writable = yes` — Allows file creation and modification by the connecting user.
- `create mask = 0777` — Assigns full permissions to any newly created files.
- `directory mask = 0777` — Assigns full permissions to any newly created directories.

## RPC-Based Enumeration

Establishing a null session to query the Remote Procedure Call (RPC) interface for system and domain data.

Establish an anonymous RPC session

```
rpcclient -U "" <TARGET_IP>
```

Retrieve server information

```
srvinfo
```

Enumerate all available shares

```
netshareenumall
```

Query specific share details

```
netsharegetinfo <SHARE_NAME>
```

> ⚠️ Gap: `rpcclient` commands may be restricted based on the user's permissions, potentially returning empty results if the session lacks sufficient privileges.

## User and Group Enumeration

Extracting valid usernames and SIDs for subsequent password attacks or RID cycling.

Enumerate domain users via RPC

```
enumdomusers
```

Query specific user details via RID

```
queryuser <RID>
```

Automated user and group extraction via Samr

```
samrdump.py <TARGET_IP>
```

### RID Brute Forcing

User enumeration is restricted but `queryuser` is allowed via RID.

Bash loop to brute force RIDs from 500 to 1100

```
for i in $(seq 500 1100); do rpcclient -N -U "" <TARGET_IP> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name|user_rid|group_rid" && echo ""; done
```

- **Account Lockout** — Brute forcing RIDs is generally safe, but using the resulting names for password guessing can trigger lockout policies found in `domain_password_information`.

## Samba Configuration Review

Assessing the `/etc/samba/smb.conf` file or server status when local or administrative access is achieved.

Check active SMB connections and Samba version

```
smbstatus
```

Filter the configuration file for active settings

```
cat /etc/samba/smb.conf | grep -v "#|;"
```

### Dangerous / misconfigured settings

- `enable privileges = yes` — Honors privileges assigned to specific SIDs.
- `logon script = <FILE_PATH>` — Script executed automatically on user login.
- `unix password sync = yes` — Synchronizes UNIX passwords with SMB passwords.