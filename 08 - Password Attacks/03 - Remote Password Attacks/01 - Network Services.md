### Network Services - Remote Management & File Sharing

#### **WinRM (Windows Remote Management)**

**Context:** Use for remote management of Windows systems via CLI when **RDP** or **SSH** are unavailable or unnecessary.

- **Ports:** 5985 (HTTP), 5986 (HTTPS).
- **Protocol:** Based on XML/SOAP; manages communication between WBEM and WMI.

**Operational Workflow: Brute Forcing and Access**

1. **Identify valid credentials:** Use **NetExec** to perform password attacks against the WinRM service.
2. **Verify "Pwn3d!" status:** If the output displays `(Pwn3d!)`, the user likely has permissions to execute system commands.
3. **Establish a shell:** Use **Evil-WinRM** to initialize a PowerShell session via **MS-PSRP**.

**Command Reference**

|Tool|Goal|Command|
|:--|:--|:--|
|**NetExec**|Brute force WinRM credentials|`netexec winrm <TARGET_IP> -u <USERNAME_LIST> -p <PASSWORD_LIST>`|
|**Evil-WinRM**|Establish interactive PowerShell shell|`evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>`|

---

#### **SSH (Secure Shell)**

**Context:** Leading service for Linux management; also utilized for connecting to remote hosts to execute commands or transfer files.

- **Port:** 22 (Default).
- **Security:** Uses symmetric/asymmetric encryption and hashing for message authenticity.

**Authentication Implications**

- **Public/Private Keys:** If an attacker obtains a **private key** that is not password protected, they can log in without credentials.

**Command Reference**

|Tool|Goal|Command|
|:--|:--|:--|
|**Hydra**|Brute force SSH credentials|`hydra -L <USERNAME_LIST> -P <PASSWORD_LIST> ssh://<TARGET_IP>`|
|**OpenSSH**|Connect to target via SSH|`ssh <USERNAME>@<TARGET_IP>`|

---

#### **RDP (Remote Desktop Protocol)**

**Context:** Use for GUI-based management of Windows systems.

- **Port:** 3389 (TCP/UDP).
- **Functionality:** Allows exchange of image, sound, and local resource sharing (printers/drives).

**Command Reference**

|Tool|Goal|Command|
|:--|:--|:--|
|**Hydra**|Brute force RDP credentials|`hydra -L <USERNAME_LIST> -P <PASSWORD_LIST> rdp://<TARGET_IP>`|
|**xFreeRDP**|Launch GUI remote desktop session|`xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>`|

---

#### **SMB (Server Message Block)**

**Context:** Responsible for file/directory sharing and printing in local networks.

- **Port:** 445.
- **Implications:** Unlocks access to shared data and potential command execution if administrative shares (C$, ADMIN$) are accessible.

**Operational Workflow: Enumeration and Access**

1. **Credential Validation:** Brute force credentials using **Hydra** or **Metasploit**.
2. **Share Enumeration:** Use **NetExec** to list available shares and check specific **Permissions** (READ/WRITE).
3. **File Interaction:** Use **smbclient** to browse, upload, or download content from identified shares.

**Command Reference**

|Tool|Goal|Command|
|:--|:--|:--|
|**Hydra**|Brute force SMB (Note: May fail on SMBv3)|`hydra -L <USERNAME_LIST> -P <PASSWORD_LIST> smb://<TARGET_IP>`|
|**Metasploit**|Brute force SMB (**Fallback for SMBv3**)|`msfconsole -q; use auxiliary/scanner/smb/smb_login; set user_file <USERNAME_LIST>; set pass_file <PASSWORD_LIST>; set rhosts <TARGET_IP>; run`|
|**NetExec**|List accessible SMB shares|`netexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --shares`|
|**smbclient**|Connect to a specific share|`smbclient -U <USERNAME> \\\\<TARGET_IP>\\<SHARENAME>`|

---

#### **Tool Installation Reference**

|Tool|Method|Command|
|:--|:--|:--|
|**NetExec**|apt|`sudo apt-get -y install netexec`|
|**Evil-WinRM**|gem|`sudo gem install evil-winrm`|

---

#### **Edge Cases & Failure Conditions**

|Scenario|Impact|Mitigation|
|:--|:--|:--|
|**SMBv3 Replies**|Outdated Hydra versions receive `invalid reply`.|Use **Metasploit `smb_login`** or update/recompile Hydra.|
|**RDP Performance**|Servers often drop parallel connections.|Reduce tasks using `-t 1` or `-t 4` and use `-W` to wait between attempts.|
|**SSH Parallelism**|Configurations may limit parallel tasks.|Reduce tasks to `-t 4`.|

#### **Dangerous/Misconfigured Settings**

| Setting                      | Security Risk                                                                                |
| :--------------------------- | :------------------------------------------------------------------------------------------- |
| **Unprotected Private Keys** | Allows immediate passwordless login if the key file is compromised.                          |
| **Default Settings**         | Services often ship with default configurations and basic authentication mechanisms.         |
| **Manual WinRM Config**      | Activation and security (certificates) depend entirely on local/domain environment security. |