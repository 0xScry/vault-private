## Password Spraying

Identifying environments where accounts are initialized with standard passwords like `ChangeMe123!` across the user base.

Execute a single-password spray against a user list over SMB to find accounts that haven't updated their setup credentials

```
netexec smb <TARGET_IP> -u <FILE_PATH> -p '<PASSWORD>'
```

- Burp Suite -> Web applications -> Prefer for HTTP-based login interfaces
- NetExec -> `netexec smb <TARGET_IP> -u <USER_LIST> -p <PASS>` -> Prefer for SMB and Active Directory environments
- Kerbrute -> Active Directory -> Prefer for targeted AD account enumeration and spraying

> ⚠️ Gap: Source omits **account lockout thresholds**. Spraying without verifying the lockout policy or implementing delays will result in mass account lockouts and immediate detection.

## Credential Stuffing

Leveraging credentials obtained from external database leaks to test for password reuse across different enterprise platforms.

Test a combined list of `username:password` pairs against an SSH service

```
hydra -C <FILE_PATH> ssh://<TARGET_IP>
```

## Default Credential Auditing

Targeting unconfigured or factory-reset devices like routers, firewalls, and databases.

Install the utility to query the Default Credentials Cheat Sheet database

```
pip3 install defaultcreds-cheat-sheet
```

Search for credentials associated with a specific vendor or product

```
creds search <SERVICE_NAME>
```

Test custom default lists compiled from vendor documentation or online research

```
hydra -C <FILE_PATH> ssh://<TARGET_IP>
```

**Internal testing environments** are high-value targets because they frequently retain default settings even when production systems are hardened.