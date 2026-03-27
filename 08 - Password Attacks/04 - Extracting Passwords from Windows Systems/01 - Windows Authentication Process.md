# Windows Authentication and Credential Storage

### Windows Authentication Architecture

The Windows authentication process is a modular system involving several protected subsystems and processes that coordinate to verify user identity and manage local security.

#### Local Security Authority (LSA)

The **LSA** is a protected subsystem responsible for:

- Authenticating users and managing local logins.
- Overseeing local security and managing security policies.
- Translating between usernames and **Security Identifiers (SIDs)**.
- Generating security audit messages and performing permission checks.

#### Interactive Logon Workflow

When a user logs into a local system, the following operational flow occurs:

1. **Input Interception**: **WinLogon** intercepts login requests from the keyboard via RPC messages from `Win32k.sys`.
2. **UI Presentation**: WinLogon launches **LogonUI** to present the graphical interface.
3. **Credential Collection**: WinLogon uses **Credential Providers** (COM objects implemented as DLLs) to obtain the account name and password.
4. **Authentication Handover**: WinLogon passes the collected credentials to the **Local Security Authority Subsystem Service (LSASS)** for verification.
5. **Verification**: LSASS uses **Authentication Packages** (DLLs) to check credentials against the **SAM** database or **Active Directory**.

---

### Local Security Authority Subsystem Service (LSASS)

Located at `%SystemRoot%\System32\Lsass.exe`, this service acts as the **gatekeeper** for the operating system. It enforces local security policies and forwards audit logs to the Event Log.

|Authentication Package|Description|
|:--|:--|
|**Lsasrv.dll**|Enforces security policies; acts as the security package manager. Contains the **Negotiate** function to select NTLM or Kerberos.|
|**Msv1_0.dll**|Used for **local machine logons** and interactive logins that do not require custom authentication.|
|**Samsrv.dll**|The Security Accounts Manager (SAM) interface; stores local accounts and supports APIs.|
|**Kerberos.dll**|Handles **Kerberos-based authentication**.|
|**Netlogon.dll**|Supports network-based logon services.|
|**Ntdsa.dll**|**Domain Controller only**. The Directory System Agent that manages the Active Directory database (`ntds.dit`).|

---

### Credential Storage Mechanisms

#### 1. Security Account Manager (SAM)

The SAM is a database file storing local user credentials as **LM or NTLM hashes** within the registry.

- **Path**: `%SystemRoot%\system32\config\SAM`
- **Registry Mount**: `HKLM\SAM`
- **Access Requirement**: **SYSTEM level privileges** are required to view or access this file.
- **Context**: Use for local authentication in a **Workgroup** environment. If **SYSKEY** is enabled, the SAM is partially encrypted to prevent offline cracking.

#### 2. Active Directory (NTDS.dit)

In a **Domain** environment, the Domain Controller (DC) validates credentials using the Active Directory database.

- **Path**: `%SystemRoot%\ntds.dit`
- **Context**: This file is synchronized across all DCs (except RODCs) and contains domain-wide account data.
- **Implication**: Compromising this file allows for large-scale credential extraction across the entire forest.

#### 3. Credential Manager

A built-in feature for storing credentials used for network resources, websites, and applications.

- **Storage Location**: Saved per user profile in the **Credential Locker**.
- **Path**:

```
C:\Users\<USERNAME>\AppData\Local\Microsoft\<VAULT_OR_CREDENTIALS>\
```

- **Attack Implication**: These credentials are encrypted but can be decrypted using various methods once access to the user context is obtained.