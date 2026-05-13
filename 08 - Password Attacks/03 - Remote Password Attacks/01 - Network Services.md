## WinRM Remote Management

Target is Windows, **5985 (HTTP)** or **5986 (HTTPS)** are open, and manual activation/configuration is confirmed.

Password spray or brute force against single or multiple targets

```
netexec winrm <TARGET_IP> -u <USER_LIST> -p <PASS_LIST>
```

Establish interactive PowerShell session (MS-PSRP) once valid credentials are found

```
evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>
```

- NetExec -> `netexec winrm <TARGET_IP> -u <USER> -p <PASS>` -> prefer for initial credential discovery and horizontal movement.
    
- Evil-WinRM -> `evil-winrm -i <TARGET_IP> -u <USER> -p <PASS>` -> prefer for stable shell access and command execution.
    
- Default configurations often lack restricted authentication mechanisms or required certificates.
    
- **Connection failure**: WinRM is not enabled by default on Windows 10/11 and requires manual environment configuration.
    
- **Pwn3d! label missing**: The absence of this tag in NetExec output indicates system command execution is likely restricted for that user.
    

## SSH Command Execution

Target is Linux or Windows with **TCP 22** open for command execution or file transfer.

Brute force credentials when key-based auth is not enforced

```
hydra -L <USER_LIST> -P <PASS_LIST> ssh://<TARGET_IP>
```

Connect to remote host for terminal access

```
ssh <USERNAME>@<TARGET_IP>
```

- Private keys left **without password protection** allow immediate access if the file is exfiltrated.
    
- **Dropped connections**: High thread counts trigger service protections; use `-t 4` in Hydra to maintain stability.
    

## RDP Graphical Access

Windows host requires GUI-based control or access to local storage/printers over **TCP 3389**.

Test credential validity against the RDP service

```
hydra -L <USER_LIST> -P <PASS_LIST> rdp://<TARGET_IP>
```

Initialize a GUI session from a Linux attack host

```
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
```

- xfreerdp -> `xfreerdp /v:<TARGET_IP> /u:<USER> /p:<PASS>` -> primary CLI tool for RDP interaction.
    
- Remmina -> GUI interface -> alternative for managing multiple sessions.
    
- **Service lockout**: RDP servers are sensitive to concurrent tasks; use `-t 1` or `-t 4` and add delays with `-W 1` or `-W 3`.
    
- **Permissions mismatch**: Credentials may be valid, but the account must be specifically **active for remote desktop** to allow a session.
    

## SMB File Sharing and Enumeration

Windows or Samba targets with **TCP 445** open for file system interaction.

Automated credential testing across targets

```
hydra -L <USER_LIST> -P <PASS_LIST> smb://<TARGET_IP>
```

Reliable brute force when targeting modern SMBv3 systems

```
use auxiliary/scanner/smb/smb_login
set rhosts <TARGET_IP>
set user_file <USER_LIST>
set pass_file <PASS_LIST>
run
```

List available shares and check specific access levels

```
netexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --shares
```

Connect to a specific share to upload or download files

```
smbclient -U <USERNAME> \\\\<TARGET_IP>\\<SHARE_NAME>
```

- Hydra -> `hydra ... smb://` -> use for basic legacy checks.
    
- Metasploit -> `smb_login` -> use when encountering **SMBv3** or modern Windows responses.
    
- **Invalid reply error**: Outdated Hydra versions cannot process **SMBv3** responses; switch to Metasploit if this occurs.
    
- **Parallelism failure**: SMB typically rejects multiple concurrent login tasks; Hydra defaults to 1 task for this protocol.