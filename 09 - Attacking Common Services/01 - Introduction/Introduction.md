### **Interacting with Common Services**

To successfully attack a service, you must understand its purpose, how to interact with it, and what tools are available for exploitation.

---

### **Internal File Sharing: SMB**

**Server Message Block (SMB)** is a standard protocol in Windows networks for sharing folders. CLI-based interaction is preferred over GUI for **automating routine tasks** and efficiently searching large file sets.

#### **1. Windows Command Shell (CMD) Interaction**

Use **CMD** when you need a native environment to automate IT operations or interact with the file system directly.

**Workflow: Mapping and Searching Shares**

1. **Map the network share** to a local drive letter to execute commands as if the files were local.
    
    ```
    net use n: \\<TARGET_IP>\<SHARE> /user:<USERNAME> <PASSWORD>
    ```
    
2. **Enumerate file count** to assess the scope of the data.
    
    ```
    dir n: /a-d /s /b | find /c ":"
    ```
    
3. **Search for sensitive filenames** (e.g., credentials or secrets).
    
    ```
    dir n:*cred* /s /b
    ```
    
4. **Search within file contents** for specific patterns.
    
    ```
    findstr /s /i <KEYWORD> n:*.*
    ```
    

**CMD Command Reference**

|Syntax|Description|
|:--|:--|
|`dir`|Displays files and subdirectories.|
|`/a-d`|Attribute filter: excludes directories from results.|
|`/s`|Recursive search through subdirectories.|
|`/b`|Bare format (removes headers and summaries).|
|`net use`|Connects to or disconnects from a shared resource.|

#### **2. Windows PowerShell Interaction**

**PowerShell** provides **Cmdlets**, which offer a more extensible scripting language than standard CMD.

**Workflow: Authenticated Mapping**

1. **Create a credential object** to securely manage authentication.
    
    ```
    $username = '<USERNAME>'
    $password = '<PASSWORD>'
    $secpassword = ConvertTo-SecureString $password -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
    ```
    
2. **Mount the drive** using the stored credentials.
    
    ```
    New-PSDrive -Name "N" -Root "\\<TARGET_IP>\<SHARE>" -PSProvider "FileSystem" -Credential $cred
    ```
    
3. **Search for files** using filtering and recursion.
    
    ```
    Get-ChildItem -Recurse -Path N:\ -Include *<KEYWORD>* -File
    ```
    
4. **Search for text patterns** within files using regular expressions.
    
    ```
    Get-ChildItem -Recurse -Path N:\ | Select-String "<KEYWORD>" -List
    ```
    

#### **3. Linux SMB Interaction**

Linux machines can mount SMB shares from Windows or Samba servers using `cifs-utils`.

**Operational Workflow**

1. **Install prerequisites**.
    
    ```
    sudo apt install cifs-utils
    ```
    
2. **Create a local mount point** and **attach the share**.
    
    ```
    sudo mkdir /mnt/<SHARE_NAME>
    sudo mount -t cifs -o username=<USERNAME>,password=<PASSWORD>,domain=<DOMAIN> //<TARGET_IP>/<SHARE> /mnt/<SHARE_NAME>
    ```
    
3. **Audit the share** using standard Linux utilities like `find` and `grep`.
    
    ```
    # Find specific filenames
    find /mnt/<SHARE_NAME>/ -name *<KEYWORD>*
    
    # Search file contents recursively
    grep -rn /mnt/<SHARE_NAME>/ -ie <KEYWORD>
    ```
    

---

### **Email Services**

Email interaction requires protocols for both sending (**SMTP**) and receiving (**POP3/IMAP**).

**Evolution Mail Client (Linux)** Use a mail client to interact with servers for message retrieval or delivery.

- **Installation:** `sudo apt-get install evolution`.
- **Launch Bypass:** If sandbox errors occur, use `export WEBKIT_FORCE_SANDBOX=0 && evolution`.
- **Security:** Verify supported encryption (TLS/STARTTLS) using the "Check for Supported Types" option.

---

### **Database Interaction**

Databases often contain sensitive information like user credentials. **Attack Implications:** If you gain access with high privileges, you may be able to **execute commands as the service account** (specifically in MSSQL).

#### **1. Command Line Utilities**

|Database|OS|Command|
|:--|:--|:--|
|**MSSQL**|Linux|`sqsh -S <TARGET_IP> -U <USERNAME> -P <PASSWORD>`|
|**MSSQL**|Windows|`sqlcmd -S <TARGET_IP> -U <USERNAME> -P <PASSWORD>`|
|**MySQL**|Linux|`mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>`|
|**MySQL**|Windows|`mysql.exe -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>`|

#### **2. GUI Applications**

GUI tools are useful for visual enumeration of databases and tables.

- **dbeaver:** Multi-platform tool supporting MSSQL, MySQL, and PostgreSQL.
- **Installation (Debian):** `sudo dpkg -i dbeaver-<VERSION>.deb`.
- **Required Connection Data:** Target IP, Port, Database Engine, and Credentials.

---

### **Common Service Tools Reference**

|Service|Tools|
|:--|:--|
|**SMB**|`smbclient`, `CrackMapExec`, `SMBMap`, `Impacket`, `psexec.py`, `smbexec.py`|
|**FTP**|`ftp`, `lftp`, `ncftp`, `filezilla`, `crossftp`|
|**Email**|`Thunderbird`, `Evolution`, `Claws`, `swaks`, `sendEmail`|
|**Databases**|`mssql-cli`, `mycli`, `dbeaver`, `MySQL Workbench`, `SSMS`|

**Note on Access Issues:** Connection failures may stem from version mismatches or restricted permissions. Use specific error codes to research official documentation or community solutions.