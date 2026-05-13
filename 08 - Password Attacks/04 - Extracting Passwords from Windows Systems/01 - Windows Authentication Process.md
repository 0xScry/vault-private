1. Identify if target is **Workgroup** or **Domain Joined**.
2. For local accounts on any machine, target the **SAM database**.
3. For domain-wide credentials, target **NTDS.dit** on a **Domain Controller**.
4. For per-user application or network secrets, target the **Credential Locker** in the user's profile.
5. If offline cracking **SAM**, ensure **SYSKEY** status is verified to account for additional encryption.

---

## Local SAM Database Access

Targeting local account hashes on standalone or domain-joined machines when **SYSTEM level privileges** are achieved.

Access the local registry hive for the SAM database

```
HKLM\SAM
```

Navigate to the physical SAM database file for offline exfiltration

```
%SystemRoot%\system32\config\SAM
```

- **Dangerous / misconfigured settings**
    
    - **SYSKEY** enabled: Partially encrypts the **SAM** file on disk with a system-generated key, preventing standard offline cracking without the key.
- **Gotchas**
    
    - **SYSTEM level privileges** are mandatory to view or access the **SAM** file or registry hive.

## Domain NTDS Database Access

Mass credential extraction from a **Domain Controller** to obtain all domain user hashes.

Locate the Active Directory database on a **Domain Controller**

```
%SystemRoot%\ntds.dit
```

- **Edge cases**
    
    - **Read-Only Domain Controllers (RODCs)**: These do not synchronize the **NTDS.dit** file in the same manner as standard **Domain Controllers**.
- **Gotchas**
    
    - **Ntdsa.dll** must be loaded to manage the **NTDS.dit** database; this module is **only loaded on Domain Controllers**.

## Credential Manager Vault Harvesting

Retrieving stored network, website, and application credentials saved within a specific user's profile.

Access the encrypted user credential storage path

```
C:\Users\<USERNAME>\AppData\Local\Microsoft\Credentials\
```

> ⚠️ Gap: The source identifies the path for **Credential Manager** but lacks the specific API calls or binary commands required to decrypt the vault once the files are exfiltrated.

- **Gotchas**
    - Credentials are **encrypted per user profile**, meaning access is typically restricted to the context of that specific user.