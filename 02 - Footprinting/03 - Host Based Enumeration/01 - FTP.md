# FTP and TFTP Enumeration and Exploitation

## Protocol Overview

### File Transfer Protocol (FTP)

FTP is an application layer protocol within the TCP/IP stack used for uploading and downloading files.

- **Control Channel:** Established on **TCP port 21** for sending commands and receiving status codes.
- **Data Channel:** Established on **TCP port 20** exclusively for data transmission.
- **Security:** FTP is a **clear-text** protocol, making it susceptible to sniffing under certain network conditions.

|Mode|Operation|Use Case|
|:--|:--|:--|
|**Active**|The client connects to port 21 and informs the server which client-side port to use for responses.|Standard connection when no client-side firewall is present.|
|**Passive**|The server announces a port for the client to establish the data channel.|**Use when a firewall protects the client**, as the client initiates the connection to bypass inbound blocks.|

### Trivial File Transfer Protocol (TFTP)

TFTP is a simpler, **unreliable** protocol compared to FTP.

- **Transport:** Uses **UDP** instead of TCP.
- **Authentication:** Does not support passwords or user authentication.
- **Limitations:** No directory listing functionality; access is restricted based on OS file permissions,.
- **Security:** Should only be used in local, protected networks due to lack of security features.

---

## Footprinting and Enumeration

### Nmap Scanning

Initial footprinting identifies the service, version, and potential misconfigurations using the Nmap Scripting Engine (NSE),.

1. **Update NSE Database:** Ensure scripts are current before scanning.
    
    ```
    sudo nmap --script-updatedb
    ```
    
2. **Execute Targeted Scan:** Use version detection, default scripts, and aggressive scanning to identify FTP banners and anonymous access.
    
    ```
    sudo nmap -sV -p21 -sC -A <TARGET_IP>
    ```
    
3. **Trace Script Execution:** **Use when debugging** scan results to see the specific commands and ports used by Nmap.
    
    ```
    sudo nmap -sV -p21 -sC -A <TARGET_IP> --script-trace
    ```
    

### Service Interaction

Interacting with the service via standard tools can reveal version information and system types via the **220 banner**.

|Tool|Command|Goal|
|:--|:--|:--|
|**nc**|`nc -nv <TARGET_IP> 21`|Basic service banner grabbing and interaction,.|
|**telnet**|`telnet <TARGET_IP> 21`|Alternative method for manual service interaction.|
|**openssl**|`openssl s_client -connect <TARGET_IP>:21 -starttls ftp`|**Use when FTP uses TLS/SSL** to inspect certificates for hostnames, emails, or locations,.|

---

## Operational Workflows

### 1. Anonymous Login and Information Gathering

**Scenario:** Use when a server allows public access without credentials to identify files that might assist in other attack vectors,.

1. Connect to the target using the `ftp` client.
    
    ```
    ftp <TARGET_IP>
    ```
    
2. Enter `<USERNAME>` as `anonymous` and leave the password blank.
3. Check the connection status and enabled features.
    
    ```
    ftp> status
    ```
    
4. Enable **debug** or **trace** modes to see raw server responses and command details.
    
    ```
    ftp> debug
    ftp> trace
    ```
    
5. List contents. If `ls_recurse_enable=YES` is set in the config, use `-R` to view the entire directory structure at once,.
    
    ```
    ftp> ls -R
    ```
    

### 2. Automated Recursive Download

**Scenario:** Use to quickly grab all accessible files for local inspection. **Warning:** This is noisy and may trigger security alarms.

1. Use `wget` to mirror the FTP directory locally.
    
    ```
    wget -m --no-passive ftp://anonymous:anonymous@<TARGET_IP>
    ```
    
2. Inspect the downloaded directory structure.
    
    ```
    tree .
    ```
    

### 3. Testing Upload Permissions (Potential RCE)

**Scenario:** If a web server shares a directory with FTP, uploading a file can lead to **Remote Command Execution (RCE)** or privilege escalation,.

1. Create a test file locally.
    
    ```
    touch testupload.txt
    ```
    
2. Upload the file to the FTP server.
    
    ```
    ftp> put testupload.txt
    ```
    
3. Verify the upload was successful and check permissions.
    
    ```
    ftp> ls
    ```
    

### 4. TFTP File Transfer

**Scenario:** Used in protected environments to move files when authentication is not required.

1. Connect to the TFTP server.
    
    ```
    tftp <TARGET_IP>
    ```
    
2. Download a specific file (requires knowing the exact filename).
    
    ```
    tftp> get <FILENAME>
    ```
    

---

## Configuration Auditing

### vsFTPd Dangerous Settings

Misconfigurations in `/etc/vsftpd.conf` can lead to unauthorized access or data disclosure,.

|Setting|Risk/Implication|
|:--|:--|
|`anonymous_enable=YES`|Allows anyone to login without a password.|
|`anon_upload_enable=YES`|Allows anonymous users to upload files, risking malware or web shells.|
|`anon_mkdir_write_enable=YES`|Allows anonymous users to create new directories.|
|`write_enable=YES`|Enables various write commands (STOR, DELE, etc.), potentially allowing file tampering.|
|`hide_ids=YES`|Overwrites UID/GID with "ftp" in listings, making it harder to identify file ownership,.|
|`ls_recurse_enable=YES`|Allows recursive directory listings, making full-server enumeration easier for attackers,.|

### Security Controls

- **Access Control:** The `/etc/ftpusers` file is used to explicitly deny specific local users (e.g., `root`, `john`) from logging into FTP.
- **Encryption:** `ssl_enable=YES` (not default) is required to secure the clear-text protocol.
- **Brute Force Protection:** Implementations like **fail2ban** are common to block IPs after multiple failed login attempts.