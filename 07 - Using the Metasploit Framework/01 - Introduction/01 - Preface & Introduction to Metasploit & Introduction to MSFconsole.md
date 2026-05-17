

## Engagement Lifecycle

Identify the target typology and service versions to determine the entry point before attempting exploitation. Use version data as the primary pivot for vulnerability research.

1. Enumerate services and validate versions to find **unpatched versions** or **outdated code**.
2. Conduct vulnerability research and code auditing to prepare the exploit.
3. Execute the selected module against the target.
4. Perform privilege escalation and transition to post-exploitation.
5. Establish pivoting and initiate data exfiltration.

---

## Framework Maintenance

Update the environment when tools need sharpening or new public modules must be imported. The `apt` package manager replaces the legacy `msfupdate` command.

Update the framework and all associated modules via the system package manager

```
sudo apt update && sudo apt install metasploit-framework
```

**Failure to update** modules results in missing the latest discovered and modularized exploits.

## Interface Initialization

Launch the console for centralized access to modules, sessions, and jobs during an assessment.

Standard launch with banner and statistics

```
msfconsole
```

Quiet launch to suppress the splash art and banner

```
msfconsole -q
```

- **msfconsole**
    - `msfconsole`
    - Prefer for "all-in-one" centralized control of the framework.
- **Metasploit Pro Console**
    - `msfconsole` (within Pro environment)
    - Prefer when requiring enterprise features like task chains, social engineering wizards, or VPN pivoting.

**Unpredictable tool behavior** can leave **traces of activity** on the target or leave the attacker platform with **open gates**.

## Architecture and Module Navigation

Reference base files or manually import/create modules when the framework requires customization or manual file verification.

List all available module categories including auxiliary, exploits, and payloads

```
ls /usr/share/metasploit-framework/modules
```

Access standalone command-line utilities for payload generation or reconnaissance

```
ls /usr/share/metasploit-framework/tools/
```

Locate Meterpreter and resource scripts for automation

```
ls /usr/share/metasploit-framework/scripts/
```

- **Auxiliary**: Use for enumeration and network discovery.
- **Exploits**: Use for executing code against a vulnerable target.
- **Post**: Use for activities following a successful foothold.

**Tunnel vision** occurs when relying on automated tools as a backbone, which **limits actions** and prevents identifying abstract security objects.

> ⚠️ Gap: The source mentions using `msfconsole` for "web app testing" in the Pro version but provides no specific command syntax or flags for the integrated `wmap` plugin or web fuzzing modules.