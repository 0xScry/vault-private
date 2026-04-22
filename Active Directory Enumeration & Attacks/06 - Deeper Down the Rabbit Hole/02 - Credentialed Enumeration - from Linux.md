# Credentialed Enumeration - Linux

Credentialed enumeration is performed after gaining a foothold via a cleartext password, NTLM hash, or SYSTEM access on a domain-joined host. This phase identifies domain attributes, group memberships, GPOs, permissions, and trusts to find escalation paths.

## 1. CrackMapExec (NetExec)

**CrackMapExec (CME)** is a primary toolset for assessing Active Directory environments using protocols like **SMB, MSSQL, SSH, and WinRM**.

### Command Reference: SMB Protocol

|Parameter|Description|
|:--|:--|
|`--users`|Lists all domain users and attributes like `badPwdCount`.|
|`--groups`|Lists domain groups and the number of members in each.|
|`--loggedon-users`|Identifies active sessions on a target host.|
|`--shares`|Enumerates available shares and access levels (READ/WRITE).|
|`-M spider_plus`|Spiders shares and logs all readable files to a JSON report.|

### Operational Workflows

**User & Group Discovery** Use this to map the attack surface and identify high-value targets.

1. **Identify** users with a `badPwdCount` of 0 to safely perform targeted password spraying without triggering lockouts.
2. **Review** group memberships to find **Administrators, Domain Admins, Executives**, or IT admin groups for privilege escalation.

```
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --users
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --groups
```

**User Hunting & Admin Validation** Use this to find where privileged users are active or to verify administrative access.

- **Attack Implication:** The `(Pwn3d!)` tag indicates the current user has **local administrator** rights on the host.
- If a **Domain Admin** (e.g., `svc_qualys`) is logged in, you can target the host to steal credentials from memory or impersonate the session.

```
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --loggedon-users
```

**Share Pillaging** Use to find sensitive data (passwords, PII, or configuration scripts) across network shares.

```
# Enumerate permissions across all shares
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --shares

# Spider a specific share for sensitive files
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> -M spider_plus --share '<SHARE_NAME>'
```

---

## 2. SMBMap

**SMBMap** is used for recursive share enumeration and file management from Linux.

### Operational Workflow

1. **Permission Check:** Identify which shares are accessible.
2. **Recursive Search:** Map the directory structure without listing every file to identify interesting subdirectories (e.g., `Accounting` or `IT`).

```
# Check share access
smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <TARGET_IP>

# Recursive directory listing (directories only)
smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <TARGET_IP> -R '<SHARE_NAME>' --dir-only
```

---

## 3. rpcclient

**rpcclient** interacts with MS-RPC via Samba to manage or enumerate AD objects. It is particularly effective if **SMB NULL sessions** are permitted.

### RID Enumeration Technique

The **Relative Identifier (RID)** is a unique hex value Windows uses to track objects. The built-in **Administrator** always has RID `0x1f4` (500).

1. **Enumerate** all domain users to map names to RIDs.
2. **Query** specific RIDs to gather detailed account information.

```
# Connect to target (supports NULL session with -N and empty quotes)
rpcclient -U "<USERNAME>" <TARGET_IP>

# Commands within the rpcclient prompt:
enumdomusers
queryuser <RID>
```

---

## 4. Impacket Toolkit

Impacket provides Python-based tools for remote execution once local administrator credentials are obtained.

|Tool|Execution Method|Context & Stealth|
|:--|:--|:--|
|**psexec.py**|Uploads an executable to `ADMIN$`; creates a service.|Provides **SYSTEM** shell; noisy and easily detectable.|
|**wmiexec.py**|Executes via WMI; semi-interactive shell.|Runs in **user context**; stealthier (no files dropped).|

```
# Gain SYSTEM shell (Noisy)
psexec.py <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>

# Gain User-context shell (Stealthier)
wmiexec.py <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

---

## 5. Windapsearch

**Windapsearch** utilizes LDAP queries to enumerate domain users, groups, and computers.

### Identifying Hidden Privileges

- **`--da`**: Quickly lists members of the **Domain Admins** group.
- **`-PU`**: Finds **privileged users** by performing recursive lookups for **nested group memberships**.
- **Attack Implication:** Nested memberships often grant users excess privileges that go unnoticed by standard audits.

```
# List Domain Admins
python3 windapsearch.py --dc-ip <TARGET_IP> -u <USERNAME>@<DOMAIN> -p <PASSWORD> --da

# Recursive search for all privileged users
python3 windapsearch.py --dc-ip <TARGET_IP> -u <USERNAME>@<DOMAIN> -p <PASSWORD> -PU
```

---

## 6. BloodHound.py

**BloodHound** uses graph theory to visualize attack paths (e.g., shortest path to Domain Admin) that are otherwise impossible to detect manually. Use the Python ingestor when you cannot access a Windows host to run SharpHound.

### Operational Workflow

1. **Collection:** Run the ingestor with the `-c all` flag to gather users, groups, GPOs, and ACLs into JSON files.
2. **Database Setup:** Start the Neo4j database service.
3. **Analysis:** Upload the zipped JSON files to the BloodHound GUI and run pre-built queries like **"Find Shortest Paths to Domain Admins"**.

```
# Collect all domain data
sudo bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -ns <TARGET_IP> -d <DOMAIN> -c all

# Start the database and GUI
sudo neo4j start
bloodhound
```

**Attack Implication:** Uncovers logical relationships between users, hosts, and ACLs to plan lateral movement and domain takeover.















