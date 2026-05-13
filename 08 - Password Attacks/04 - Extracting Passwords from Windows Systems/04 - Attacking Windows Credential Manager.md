## Credential Manager Enumeration

Targeting stored login tokens for OneDrive, domain resources, or web credentials when in a user session,.

List all stored credentials for the current user via CLI

```
cmdkey /list
```

Open the Stored User Names and Passwords GUI for manual export or management

```
rundll32 keymgr.dll,KRShowKeyMgr
```

- **Gotchas**
    - **virtualapp/didlogical** entries are internal Microsoft account IDs and should be ignored during triage.
    - **Exported .crd files** require a user-supplied password during creation which must be known to import the backup on another system.

## Stored Credential Impersonation

`cmdkey` output identifies `Domain:interactive` credentials for a target user, allowing for lateral movement.

Spawn a shell as the stored user using existing cached credentials

```
runas /savecred /user:<DOMAIN>\<USERNAME> cmd
```

- **Gotchas**
    - **Local machine persistence** must be verified in `cmdkey` output to ensure credentials survive a reboot.

## Credential Manager Extraction

Local admin or SYSTEM access is available and clear-text credentials for external services (e.g., OneDrive) are required from memory,.

Dump credentials from the LSASS process using Mimikatz

```
privilege::debug
sekurlsa::credman
```

- **Tool comparison**
    
    - Mimikatz → `sekurlsa::credman` → Prefer for live memory extraction from LSASS,.
    - Mimikatz → `dpapi` module → Prefer for manual decryption of vault files on disk.
    - SharpDPAPI / DonPAPI / LaZagne → External binaries → Use for automated DPAPI-protected store enumeration and extraction.
- **Dangerous / misconfigured settings**
    
    - **Disabled Credential Guard**: Systems without Virtualization-based Security (VBS) fail to protect DPAPI master keys in secured memory enclaves.
- **Gotchas**
    
    - **Credential Guard** prevents standard memory dumping tools from accessing DPAPI master keys by isolating them in secured enclaves.

> ⚠️ Gap: The source indicates `sekurlsa::credman` targets LSASS, but if **Credential Guard** is active, this technique will fail to retrieve the keys necessary to decrypt the vault as they are stored in a VBS-isolated memory space rather than standard LSASS memory.