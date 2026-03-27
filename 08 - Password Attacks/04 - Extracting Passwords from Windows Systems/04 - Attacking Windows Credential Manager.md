# Attacking Windows Credential Manager & Windows Vault

The **Windows Credential Manager** (and its underlying encrypted stores known as **Credential Lockers** or **Windows Vaults**) allows users and applications to store credentials for websites, network resources, and domain accounts. These are stored in encrypted folders under user and system profiles, protected by **DPAPI** and, in newer versions, **Credential Guard**.

### Credential Types

|Name|Description|
|:--|:--|
|**Web Credentials**|Associated with websites and online accounts; used by Internet Explorer and legacy Microsoft Edge.|
|**Windows Credentials**|Login tokens for services (e.g., OneDrive), domain users, local network resources, and shared directories.|

---

### Phase 1: Enumeration

Before attempting extraction, identify what credentials are saved in the current user's profile to determine if they are worth targeting for lateral movement or impersonation.

1. **List stored credentials** using the native `cmdkey` utility to view targets, types, and persistence.
2. **Identify high-value targets**: Look for `Domain Password` or `Generic` types. Ignore internal IDs like `virtualapp/didlogical`.
3. **Check for Interactive credentials**: Entries formatted as `Domain:interactive=<DOMAIN>\<USERNAME>` indicate the credential can be used for logon sessions.

**Command Reference:**

|Command|Goal|
|:--|:--|
|`cmdkey /list`|**Enumerate** all credentials stored for the current user profile.|
|`rundll32 keymgr.dll,KRShowKeyMgr`|Open the **Stored User Names and Passwords** window to manually view or edit entries.|

---

### Phase 2: Exploitation via User Impersonation

When a credential is marked as **Interactive**, you can impersonate that user without knowing their cleartext password by leveraging the stored session.

- **When to use**: Use this when you find a stored domain credential and need to execute commands in the context of that user.
- **Implication**: This unlocks a new command shell as the target user.

```
runas /savecred /user:<DOMAIN>\<USERNAME> cmd
```

_Note: The `/savecred` flag instructs the system to use the existing stored credential._

---

### Phase 3: Credential Extraction

For full extraction of cleartext passwords or secrets, use specialized tools to dump data from memory or decrypt vault files.

#### Methodology: Memory Injection (Mimikatz)

This technique targets the **LSASS** process to pull credentials directly from the `sekurlsa` module.

1. **Gain debug privileges** to allow the tool to interact with system processes.
2. **Dump Credential Manager secrets** from memory.

```
# Start Mimikatz
.\mimikatz.exe

# Elevate privileges
privilege::debug

# Extract credentials from memory
sekurlsa::credman
```

#### Alternative Extraction Tools

If Mimikatz is flagged or unavailable, the following tools can also be used for enumeration and extraction:

- **SharpDPAPI**
- **LaZagne**
- **DonPAPI**

---

### Operational Security & Edge Cases

- **Credential Guard**: In newer Windows versions, DPAPI master keys may be stored in secured memory enclaves (Virtualization-based Security), making memory-based extraction significantly harder.
- **Manual Backups**: Users can export vaults to `.crd` files via the Control Panel. These are encrypted with a user-supplied password and can be moved to other systems.
- **Persistence**: Credentials marked with **Local machine persistence** survive reboots, making them reliable targets for long-term access.