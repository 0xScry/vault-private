# Windows Credential Hunting

**Credential hunting** is the process of searching the file system and applications to discover stored or documented credentials after gaining initial access. Success relies on tailoring the search to the **target system's role** (e.g., an IT admin's workstation vs. a Windows server) to reduce random guessing.

### 1. Manual GUI Search

Use when **GUI access** (like RDP) is available to leverage built-in search indexing.

- **Goal:** Find files containing passwords or documentation created by the user.
- **Action:** Use the **Windows Search bar** with key terms such as `pass`.
- **Impact:** May reveal "Change your password" settings or text files containing cleartext credentials.

---

### 2. Automated Discovery with LaZagne

Use to quickly extract insecurely stored credentials from browsers, chat logs, and system configurations.

#### LaZagne Module Reference

|Module|Targets|
|:--|:--|
|**browsers**|Extracts from Chromium, Firefox, Edge, and Opera.|
|**chats**|Extracts from applications like Skype.|
|**mails**|Searches mailboxes like Outlook and Thunderbird.|
|**memory**|Dumps passwords from KeePass and LSASS.|
|**sysadmin**|Extracts from tools like OpenVPN and WinSCP.|
|**windows**|Targets LSA secrets, Credential Manager, etc.|
|**wifi**|Dumps stored WiFi credentials.|

#### Operational Workflow

1. **Transfer** the standalone `LaZagne.exe` from the `<ATTACK_IP>` to the target.
    - _Note:_ If using **xfreerdp**, you can copy and paste the file directly into the RDP session.
2. **Open** a command prompt or PowerShell session and navigate to the upload directory.
3. **Execute** the tool to run all modules:

```
start LaZagne.exe all
```

- **Verbose Mode:** Use `-vv` to monitor the background activity and specific software being targeted.
- **Attack Implication:** Many applications store credentials insecurely; LaZagne can automate the decryption of these databases, providing cleartext passwords for lateral movement.

---

### 3. CLI Pattern Searching (findstr)

Use when only **CLI access** is available or to perform broad searches across specific configuration and script file types.

- **Goal:** Search for the string "password" across common configuration and script extensions.
- **Action:** Execute the following command from the root of the target drive:

```
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

#### Command Parameters

|Parameter|Description|
|:--|:--|
|**/S**|Searches the current directory and all subdirectories.|
|**/I**|Specifies that the search is not case-sensitive.|
|**/M**|Prints only the filename if a file contains a match.|
|**/C**|Uses the specified string as a literal search phrase.|

---

### Strategy & Decision Points

- **System Context:** Focus on tools and terms relevant to the machine's function. IT admin workstations are high-value targets for finding documentation or admin tool credentials.
- **Browser Storage:** Browsers like Chrome and Firefox are primary targets because they offer built-in credential storage. While encrypted, tools like `firefox_decrypt` or `LaZagne` can often recover them.
- **Insecure Storage:** Many sysadmin tools (e.g., WinSCP) may store credentials in cleartext or easily decryptable formats within configuration files.