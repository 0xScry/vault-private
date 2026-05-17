## Local GUI Discovery

Land on a workstation, specifically one used by an IT admin, where documented passwords or default credentials likely exist in local files.

Search keywords in the Windows Search bar to identify sensitive system settings and local documentation. `password` `login` `admin` `pass`

## Automated Application Credential Extraction

Targeting browser databases, chat logs, mailboxes, and sysadmin configurations for insecurely stored cleartext.

Run all modules with verbose output to identify background attempts and capture credentials from 35+ browsers and tools like WinSCP.

```
start LaZagne.exe all -vv
```

- LaZagne -> `start LaZagne.exe all` -> preferred for broad coverage including LSA secrets, browsers, and sysadmin tools.
- `firefox_decrypt` / `decrypt-chrome-passwords` -> standalone execution -> preferred when **encrypted browser storage** prevents LaZagne from recovering cleartext.

**Encrypted credential databases** in modern browsers like Chrome, Edge, and Firefox require specific decryption tools if automated tools fail.

**Cleartext storage** in applications like WinSCP allows for immediate credential recovery without further decryption.

## Command Line Pattern Searching

Hunting for plaintext strings across configuration, script, and log files when GUI access is restricted or deep directory traversal is required.

Execute recursive, case-insensitive string matching across common sensitive file extensions.

```
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

> ⚠️ Gap: `findstr` will silently fail to return results from files where the current user lacks read permissions; elevate privileges or target user-writable directories to ensure full coverage.