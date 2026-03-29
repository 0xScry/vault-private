# SMB/Samba Enumeration and Configuration Notes

**SMB (Server Message Block)** is a client-server protocol used to regulate access to files, directories, and network resources like printers. **Samba** is the open-source implementation allowing Unix-based systems to communicate via SMB/CIFS with Windows systems.

### Protocol Fundamentals

|Version|Supported OS|Key Features|
|:--|:--|:--|
|**CIFS**|Windows NT 4.0|Communication via NetBIOS|
|**SMB 1.0**|Windows 2000|Direct connection via TCP|
|**SMB 2.0**|Windows Vista / Server 2008|Performance upgrades, message signing, caching|
|**SMB 2.1**|Windows 7 / Server 2008 R2|Locking mechanisms|
|**SMB 3.0**|Windows 8 / Server 2012|Multichannel connections, end-to-end encryption|
|**SMB 3.1.1**|Windows 10 / Server 2016|Integrity checking, AES-128 encryption|

**Network Ports:**

- **TCP 137, 138, 139:** Used when SMB commands are transmitted over **Samba** to older **NetBIOS** services.
- **TCP 445:** Used exclusively by **CIFS** and modern SMB versions for direct TCP communication.

---

### Samba Configuration & Dangerous Settings

Samba settings are defined in `/etc/samba/smb.conf`. Global settings apply to all shares but can be overridden by specific share configurations.

#### Dangerous/Misconfigured Settings

Misconfigurations often occur when settings intended for testing are left in production, granting attackers visibility or write access.

|Setting|Attack Implication|
|:--|:--|
|`browseable = yes`|Allows attackers to list and see the share in the directory.|
|`read only = no`|Allows attackers to modify existing files or upload malicious ones.|
|`writable = yes`|Explicitly allows users to create and modify files.|
|`guest ok = yes`|Allows connection to the service without a password.|
|`enable privileges = yes`|Honors privileges assigned to specific SIDs.|
|`create mask = 0777`|Newly created files receive full global permissions.|
|`directory mask = 0777`|Newly created directories receive full global permissions.|
|`logon script = <SCRIPT>`|Script executed on user login; potential for persistence or code execution.|
|`magic script = <SCRIPT>`|Script executed when closed; potential for code execution.|

---

### Enumeration Methodology

#### 1. Service Footprinting

Use **Nmap** to identify the service version and basic security configurations.

- **Goal:** Determine if the target is running Samba and identify the version for known vulnerability research.

```
sudo nmap <TARGET_IP> -sV -sC -p139,445
```

#### 2. Anonymous Share Enumeration

Attempt to list shares using a **null session** (anonymous access without a valid username or password).

- **Goal:** Identify accessible shares that might contain sensitive data or allow write access.

```
# List shares anonymously
smbclient -N -L //<TARGET_IP>
```

#### 3. Interactive Share Access

If a share is identified (e.g., `notes`), connect to it to inspect the contents.

- **Goal:** Download interesting files or explore the directory structure.

```
# Connect to a specific share
smbclient //<TARGET_IP>/<SHARE_NAME>
```

**Common Interactive Commands:**

- `ls`: List files in the current share directory.
- `get <FILE>`: Download a file to your attack machine.
- `!<COMMAND>`: Execute a local system command on your attack machine without disconnecting (e.g., `!ls`, `!cat`).

#### 4. RPC Enumeration (rpcclient)

Use **rpcclient** to interact with MS-RPC functions to leak information about the domain, shares, and users.

```
# Connect with a null session
rpcclient -U "" <TARGET_IP>
```

**Key rpcclient Queries:**

|Command|Goal|
|:--|:--|
|`srvinfo`|Retrieve server information.|
|`enumdomains`|Enumerate all deployed domains.|
|`netshareenumall`|List all available shares.|
|`netsharegetinfo <SHARE>`|Get detailed permissions and paths for a specific share.|
|`enumdomusers`|Enumerate all domain users.|
|`queryuser <RID>`|Get detailed information (like group RIDs) for a specific user.|

#### 5. User RID Brute Forcing

If `enumdomusers` is restricted, brute force **Relative Identifiers (RIDs)** to discover valid usernames.

- **Goal:** Identify usernames for future password attacks or to find administrative accounts.

**Manual Bash Loop:**

```
for i in $(seq 500 1100); do
  rpcclient -N -U "" <TARGET_IP> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name|user_rid|group_rid" && echo "";
done
```

**Automated RID Dumping (Impacket):**

```
samrdump.py <TARGET_IP>
```

#### 6. Comprehensive Automated Enumeration

Use specialized tools to automate the collection of OS information, share permissions, and password policies.

- **Goal:** Quickly gather a broad overview of the target's SMB environment.

```
# Check share permissions with CrackMapExec
crackmapexec smb <TARGET_IP> --shares -u '' -p ''

# Map shares with SMBMap
smbmap -H <TARGET_IP>

# Full automated enumeration with enum4linux-ng
./enum4linux-ng.py <TARGET_IP> -A
```

### Administrative Monitoring

To view active SMB connections from the server-side, use `smbstatus`. This reveals which users are connected, their source IP, and which shares they are accessing.

```
# Run on the SMB server
smbstatus
```