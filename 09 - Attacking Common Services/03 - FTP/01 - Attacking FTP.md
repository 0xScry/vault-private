# FTP (File Transfer Protocol) Service Attacks

The File Transfer Protocol (FTP) facilitates file transfers and directory operations over **TCP port 21**. Attacks typically focus on abusing misconfigurations, excessive privileges, or protocol-specific behaviors.

## 1. Initial Enumeration

The primary goal is to identify the service version and check for **anonymous authentication**.

|Command|Description|
|:--|:--|
|`sudo nmap -sC -sV -p 21 <TARGET_IP>`|Runs default scripts (`-sC`) to check for anonymous login and version detection (`-sV`).|
|`nc -nv <TARGET_IP> 21`|Manually interacts with the service to grab the banner.|

**Why it matters:** The FTP banner often reveals the specific software version, which helps in identifying known vulnerabilities.

---

## 2. Exploiting Anonymous Authentication

Use when a server allows login with the username `anonymous` and no password.

### Operational Workflow

1. **Connect** to the server:
    
    ```
    ftp <TARGET_IP>
    ```
    
2. **Authenticate** using `<USERNAME>` as `anonymous` and leave the password field blank.
3. **Explore** the directory structure using `ls` and `cd`.
4. **Transfer** files based on discovered permissions.

### Command Reference: File Operations

|Command|Action|Goal|
|:--|:--|:--|
|`get <FILE>`|Download|Retrieve sensitive configuration or credential files.|
|`mget *.txt`|Download Multiple|Efficiently pull all files of a specific type.|
|`put <FILE>`|Upload|Transfer malicious scripts to the server.|
|`mput *.php`|Upload Multiple|Upload web shells or tools in bulk.|

**Attack Implications:**

- **Data Leakage:** Companies may inadvertently store sensitive information in folders accessible to anonymous users.
- **Remote Code Execution (RCE):** If write permissions are enabled, an attacker can upload scripts (e.g., PHP). If the file path is discoverable (e.g., via path traversal in a web app), the script can be executed.

---

## 3. Credential Brute Forcing

Use when anonymous authentication is disabled and you have a list of potential usernames or passwords.

### Brute Forcing with Medusa

**Why it matters:** While effective against weak credentials, modern applications often have account lockout protections. **Password spraying** is generally more effective.

|Parameter|Description|
|:--|:--|
|`-u <USERNAME>`|Target a specific single user.|
|`-U <USER_LIST>`|Path to a file containing multiple usernames.|
|`-P <PASS_LIST>`|Path to a file containing a list of passwords.|
|`-M ftp`|Specifies the protocol (FTP).|
|`-h <TARGET_IP>`|The target hostname or IP address.|

**Command:**

```
medusa -u <USERNAME> -P <PASS_LIST> -h <TARGET_IP> -M ftp
```

---

## 4. FTP Bounce Attack

This technique uses an FTP server as an intermediary to send outbound traffic to another internal device, effectively bypassing firewall restrictions.

**Scenario Context:** Use when a target host (`<TARGET_IP>`) is not exposed to the internet but is reachable by an FTP server in the DMZ (`<PIVOT_IP>`).

### Operational Workflow

1. **Identify** a vulnerable FTP server that allows the `PORT` command to point to a different host than the client.
2. **Execute** a bounce scan using Nmap to discover open ports on the internal target:
    
    ```
    nmap -Pn -v -n -p <PORT> -b <USERNAME>:<PASSWORD>@<PIVOT_IP> <TARGET_IP>
    ```
    

**Attack Implications:** This unlocks the ability to scan and potentially attack internal infrastructure that is otherwise unreachable from the attack machine. **Modern FTP servers** usually have protections against this by default, but misconfigurations can still leave them vulnerable.

---

## 5. Critical Misconfigurations

The following settings frequently lead to service compromise:

|Misconfiguration|Impact|
|:--|:--|
|**Anonymous Login Enabled**|Allows unauthorized access to the filesystem and potential data theft.|
|**Write Permissions for Anonymous**|Enables attackers to upload malicious scripts or web shells.|
|**PORT Command Unrestricted**|Enables the FTP Bounce Attack to proxy traffic into internal networks.|
|**Weak Password Policy**|Facilitates successful brute forcing or password spraying.|