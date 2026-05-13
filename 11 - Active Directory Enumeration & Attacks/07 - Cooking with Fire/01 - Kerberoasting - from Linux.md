## Kerberoasting Enumeration

SPNs are identified in the domain and you possess a domain user's cleartext password, NTLM hash, or a shell in a domain user context.

List all SPN accounts associated with the domain to identify high-privilege targets like **Domain Admins**

```
GetUserSPNs.py -dc-ip <DC_IP> <DOMAIN>/<USERNAME>
```

- **Dangerous / misconfigured settings**
    
    - Service accounts added to privileged groups like **Domain Admins** directly or via nested membership.
    - Domain accounts used for services to bypass network authentication limits of built-in accounts.
- **Gotchas**
    
    - **Unknown Domain Controller IP** prevents querying the domain for SPN listings.

## TGS Ticket Acquisition

High-privilege SPNs are confirmed and you need to retrieve encrypted tickets for offline brute-forcing.

Request all available TGS tickets for the domain and output to terminal

```
GetUserSPNs.py -dc-ip <DC_IP> <DOMAIN>/<USERNAME> -request
```

Request a TGS ticket for a specific service account to minimize noise

```
GetUserSPNs.py -dc-ip <DC_IP> <DOMAIN>/<USERNAME> -request-user <USERNAME>
```

Request and save the TGS ticket directly to a file compatible with cracking tools

```
GetUserSPNs.py -dc-ip <DC_IP> <DOMAIN>/<USERNAME> -request-user <USERNAME> -outputfile <FILE_PATH>
```

- **Edge cases**
    
    - Targets across forest trusts can be Kerberoasted if **authentication is permitted across the trust boundary**.
- **Gotchas**
    
    - **Retrieving a ticket** does not grant immediate command execution; the ticket is merely the material for an offline attack.

## Offline Password Cracking

TGS-REP tickets have been exported and are ready for brute-force attempts on an attack host or GPU rig.

Brute-force the Kerberos 5 TGS-REP etype 23 hash using a wordlist

```
hashcat -m 13100 <FILE_PATH> <FILE_PATH>
```

- **Dangerous / misconfigured settings**
    
    - Service accounts configured with **weak or reused passwords**, or passwords identical to the username.
- **Gotchas**
    
    - **Standard cracking rigs** may fail to recover cleartext because TGS tickets take significantly longer to crack than NTLM hashes.

## Credential Validation

Cleartext passwords have been recovered from Hashcat and require verification against the domain.

Validate recovered credentials and check for administrative privileges over the DC

```
crackmapexec smb <DC_IP> -u <USERNAME> -p <PASSWORD>
```

- **Edge cases**
    
    - Cracking a low-privilege service account still allows crafting service tickets to access specific services (like MSSQL) as **sysadmin**.
- **Gotchas**
    
    - **Strong password policies** may result in zero successful cracks even after days of attempts, necessitating a lower risk rating in reports.