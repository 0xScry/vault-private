# **Password Policy Enumeration**

Identifying the **domain password policy** is a prerequisite for password spraying. This data allows you to tailor attacks to meet complexity requirements while staying under **lockout thresholds** to avoid alerting defenders or disrupting client operations.

---

### **1. Linux-Based Enumeration (Credentialed)**

When valid domain credentials are known, tools can remotely pull the full policy via SMB.

**Operational Workflow:**

1. **Authenticate** to the Domain Controller using known credentials.
2. **Request** the password policy to identify lockout limits and complexity rules.

|Tool|Command|Purpose|
|:--|:--|:--|
|**CrackMapExec**|`crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --pass-pol`|Dumps policy including history, length, and lockout settings.|
|**rpcclient**|`rpcclient -U "<USERNAME>%<PASSWORD>" <TARGET_IP>`|Establishes an authenticated RPC session to query the domain.|

---

### **2. Linux-Based Enumeration (Unauthenticated)**

Used when no credentials are available. These techniques exploit **legacy misconfigurations** or insecure upgrades where anonymous access is permitted.

#### **A. SMB NULL Sessions**

These occur when legacy Domain Controllers are upgraded without hardening, allowing unauthenticated attackers to list users and policies.

**Operational Workflow:**

1. **Verify Access:** Attempt to connect with an empty username and password.
2. **Query Domain Information:** Confirm the connection is valid and retrieve domain roles.
3. **Extract Policy:** Pull specific password and lockout attributes.

|Tool|Command|Purpose|
|:--|:--|:--|
|**rpcclient**|`rpcclient -U "" -N <TARGET_IP>`|Connects with a NULL session.|
|**rpcclient** (In session)|`querydominfo`|Confirms NULL session access and domain role.|
|**rpcclient** (In session)|`getdompwinfo`|Retrieves minimum password length and complexity flags.|
|**enum4linux**|`enum4linux -P <TARGET_IP>`|Automates policy retrieval via NULL session.|
|**enum4linux-ng**|`enum4linux-ng -P <TARGET_IP> -oA <FILENAME>`|Next-gen version; exports data to YAML/JSON for tool integration.|

#### **B. LDAP Anonymous Bind**

Used if an administrator has enabled anonymous binds to support specific application requirements, inadvertently exposing **Active Directory objects** to unauthenticated users.

|Tool|Command|Purpose|
|:--|:--|:--|
|**ldapsearch**|`ldapsearch -H ldap://<TARGET_IP> -x -b "DC=,DC=" -s sub "*" \|grep -m 1 -B 10 pwdHistoryLength`|

---

### **3. Windows-Based Enumeration**

Used when you have local access to a domain-joined host. This is preferred if **EDR** or client restrictions prevent the transfer of external tools.

**Operational Workflow:**

1. **Local Context:** Use built-in binaries to avoid detection from file transfers.
2. **Module Import:** Use PowerShell scripts for more detailed Kerberos policy data.

|Tool|Command|Purpose|
|:--|:--|:--|
|**Net.exe**|`net accounts`|Built-in binary; displays lockout thresholds and history length.|
|**Net.exe** (NULL)|`net use \\<TARGET_IP>\ipc$ "" /u:""`|Establishes a NULL session from Windows to confirm enumeration access.|
|**PowerView**|`import-module .\PowerView.ps1; Get-DomainPolicy`|Retrieves `SystemAccess` and `KerberosPolicy` details via PowerShell.|

---

### **Dangerous & Legacy Misconfigurations**

These settings allow unauthenticated discovery of sensitive domain information.

|Configuration|Risk / Attack Implication|
|:--|:--|
|**SMB NULL Session**|Allows unauthenticated listing of users, groups, and password policies.|
|**LDAP Anonymous Bind**|Permits unauthenticated LDAP requests to view all AD objects.|
|**Lockout Threshold: 0**|Accounts can be brute-forced indefinitely without locking.|
|**Threshold: 5**|Attempting **2-3 sprays every 31 minutes** is safe to avoid lockout.|

---

### **Operational Safety & Decision Logic**

If the password policy **cannot be retrieved**, adhere to these safety limits to avoid account lockouts:

- **Spraying Limit:** Run a maximum of **one or two** password spray attempts total.
- **Wait Period:** Wait **over one hour** between attempts.
- **Threshold Assumptions:** Never assume a lockout threshold of 5; some organizations set it as low as **3**, requiring manual administrator intervention to unlock.
- **Stealth:** Use built-in Windows commands like `net accounts` if landing on a target system to avoid the noise of transferring tools.