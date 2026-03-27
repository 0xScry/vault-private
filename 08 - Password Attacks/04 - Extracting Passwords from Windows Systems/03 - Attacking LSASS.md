### **Attacking LSASS (Local Security Authority Subsystem Service)**

The **LSASS** process is a core Windows component responsible for enforcing security policies, handling user authentication, and storing sensitive credential material in memory. Targeting LSASS allows for the extraction of credentials from active logon sessions.

Creating a memory dump of LSASS facilitates **offline attacks**, which provides more flexibility in attack speed and reduces the time spent on the target system.

---

### **LSASS Memory Dumping Techniques**

#### **Method 1: Task Manager (GUI)**

- **Scenario:** Use when you have an interactive graphical session on the target.
- **Workflow:**
    1. Open **Task Manager**.
    2. Locate the **Local Security Authority Process**.
    3. Right-click the process and select **Create dump file**.
    4. The file `lsass.DMP` is saved in `%temp%`; transfer this to the attack host for analysis.

#### **Method 2: Rundll32.exe & Comsvcs.dll (CLI)**

- **Scenario:** Use when only command-line access (shell) is available or a faster method is required.
- **Constraint:** Modern anti-virus (AV) tools frequently flag this method as malicious.
- **Workflow:**
    1. **Identify the LSASS Process ID (PID):** This must be done before issuing the dump command.
    2. **Generate the Dump:** Use an elevated session to call the `MiniDump` function within `comsvcs.dll`.

**Command Reference: PID Discovery**

|Environment|Command|Goal|
|:--|:--|:--|
|**CMD**|`tasklist /svc`|Locate `lsass.exe` and its PID in the output.|
|**PowerShell**|`Get-Process lsass`|View the PID in the `Id` field.|

**Command Reference: Dumping LSASS**

|Action|Command|
|:--|:--|
|**Dump Memory**|`rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> <TARGET_PATH> full`|

---

### **Offline Credential Extraction**

#### **Parsing with Pypykatz**

- **Scenario:** Use on a Linux-based attack host to extract credentials from the `.dmp` file without running tools directly on the target.
- **Implication:** Provides a "snapshot" of credentials present in memory at the time of the dump.

|Tool|Command|
|:--|:--|
|**Pypykatz**|`pypykatz lsa minidump <DUMP_PATH>`|

**Technical Impact of Extracted Data:**

|Package|Extracted Material|Attack Unlock|
|:--|:--|:--|
|**MSV**|SID, Username, NT & SHA1 hashes.|Facilitates offline cracking or local authentication.|
|**WDIGEST**|Clear-text passwords.|**Note:** Enabled by default in older Windows (XP-8, Server 2003-2012) or unpatched systems.|
|**Kerberos**|Passwords, ekeys, tickets, and pins.|Enables lateral movement to other domain-joined systems.|
|**DPAPI**|Masterkeys for logged-on users.|Used to decrypt secrets associated with various applications.|

---

### **Password Cracking**

- **Scenario:** Recover clear-text passwords from extracted NT hashes using a wordlist.

|Tool|Command|
|:--|:--|
|**Hashcat**|`sudo hashcat -m 1000 <NT_HASH> <WORDLIST_PATH>`|

**Operational Note:** In the source example, mode `1000` is used for NT hashes with a standard wordlist like `rockyou.txt`.