### **Linux Remote Management Protocols**

#### **1. Secure Shell (SSH)**

SSH is the standard for encrypted remote management on **TCP port 22**. While secure by design, misconfigurations or outdated versions (SSH-1) provide entry points for attackers.

**A. Service Fingerprinting** Use these techniques to identify the SSH version, supported authentication methods, and potential cryptographic weaknesses.

|Goal|Command|
|:--|:--|
|**Audit** server/client configuration and encryption algorithms|`./ssh-audit.py <TARGET_IP>`|
|**Enumerate** supported authentication methods (e.g., password, publickey)|`ssh -v <USERNAME>@<TARGET_IP>`|
|**Force** a specific authentication method (e.g., for brute-forcing)|`ssh -v <USERNAME>@<TARGET_IP> -o PreferredAuthentications=password`|

- **Decision Point:** If the banner indicates **Protocol 1**, the connection is vulnerable to **MITM attacks**.
- **Decision Point:** If the banner (e.g., `SSH-1.99`) indicates support for both SSH-1 and SSH-2, prioritize attacking the weaker protocol.

**B. Configuration Analysis** When auditing a compromised system, check `/etc/ssh/sshd_config` for settings that facilitate lateral movement or privilege escalation.

**Dangerous SSH Settings**

|Setting|Attack Implication|
|:--|:--|
|`PasswordAuthentication yes`|Enables password **brute-forcing** attempts.|
|`PermitEmptyPasswords yes`|Allows access without a password if the account has none.|
|`PermitRootLogin yes`|Allows direct administrative access via SSH.|
|`Protocol 1`|Vulnerable to **Man-In-The-Middle (MITM)** attacks.|
|`X11Forwarding yes`|Potential for command injection (specifically version 7.2p1).|
|`AllowTcpForwarding yes`|Enables **port forwarding** to bypass firewall restrictions.|

---

#### **2. Rsync**

Rsync (standard **TCP port 873**) is used for fast file synchronization and backups. It is a high-value target because it often contains sensitive data (like SSH keys) and may not require authentication.

**Operational Workflow: Enumerating and Syncing Files**

1. **Service Identification:** Confirm Rsync is active and identify the protocol version.
    
    ```
    sudo nmap -sV -p 873 <TARGET_IP>
    ```
    
2. **Banner Grabbing:** Manually probe the service to list available shares.
    
    ```
    nc -nv <TARGET_IP> 873
    # Once connected, type: #list
    ```
    
3. **Share Enumeration:** List the contents of a specific share without downloading them.
    
    ```
    rsync -av --list-only rsync://<TARGET_IP>/<SHARE_NAME>
    ```
    
4. **Data Exfiltration:** Sync all files from the remote share to your attack machine.
    
    ```
    rsync -av rsync://<TARGET_IP>/<SHARE_NAME>
    ```
    

- **Attack Implication:** If a `.ssh` directory is visible in the share, you may be able to pull private keys to gain direct SSH access.
- **Scenario Context:** If standard Rsync is blocked, check if it is configured to run **over SSH** using the `-e ssh` flag.

---

#### **3. R-Services**

R-Services (**TCP ports 512, 513, 514**) are legacy protocols that rely on "trusted" relationships rather than strong encryption. They are highly vulnerable to **MITM attacks** because traffic is unencrypted.

**A. Command Reference**

|Command|Port|Description|
|:--|:--|:--|
|**rcp**|514|Copy files between hosts; no warning when overwriting.|
|**rsh**|514|Opens a shell on a remote machine without a login procedure.|
|**rexec**|512|Runs shell commands; requires a username and password.|
|**rlogin**|513|Remote login (similar to telnet).|

**B. Exploiting Trust Relationships** Authentication is often bypassed by entries in **`/etc/hosts.equiv`** (global) or **`.rhosts`** (per-user).

1. **Check for Open Ports:**
    
    ```
    sudo nmap -sV -p 512,513,514 <TARGET_IP>
    ```
    
2. **Attempt Unauthenticated Login:** If the target trusts your IP or has a wildcard (`+`) configured, you can log in without a password.
    
    ```
    rlogin <TARGET_IP> -l <USERNAME>
    ```
    

**C. Post-Exploitation Enumeration** Once access is gained, use R-commands to find other active users and hosts to target for lateral movement.

|Goal|Command|
|:--|:--|
|**List** interactive sessions on the local network (UDP 513)|`rwho`|
|**Detailed** info on logged-in users (host, TTY, login time)|`rusers -al <TARGET_IP>`|

- **Decision Point:** Use the information from `rwho` and `rusers` to compile a list of **usernames and hostnames** for credential stuffing or password reuse attacks.