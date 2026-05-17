## Initial Target Enumeration

Target OS and service versions are unknown, requiring identification to map vulnerabilities to specific framework modules.

Initial enumeration to identify services like SMB on **445** or RPC on **135**

```
nmap -sC -sV -Pn <TARGET_IP>
```

- **SMB Message signing disabled** on the target is a **dangerous** default state that facilitates easier exploitation.

## Module Discovery and Navigation

Services are identified and require a searchable database to find relevant exploit, auxiliary, or post-exploitation modules.

Launch the framework console with root privileges

```
sudo msfconsole
```

Find modules associated with a specific service or vulnerability

```
search <SERVICE_NAME>
```

Select a module by its full path or the index number from the latest search results

```
use <MODULE_PATH>
```

Refresh the framework database after modifying custom modules

```
reload
```

- **Search index numbers are volatile** and will change as the maintainers update the codebase or when custom modules are imported.

## Exploit Configuration and Execution

A module is selected and requires environment-specific parameters before delivery.

Display all required and optional parameters for the current module and payload

```
options
```

Configure necessary variables such as **RHOSTS**, **LHOST**, and credentials

```
set <VARIABLE> <VALUE>
```

Launch the exploit and initiate the payload delivery sequence

```
exploit
```

- **LHOST** can be set to a specific IP or a local interface ID like `tun0`.
- **Service start timeout** may trigger a warning during execution but is not a **failure condition** if running a command or non-service executable.

## Post-Exploitation Interaction

A session is established and requires interaction for file management, process manipulation, or native command execution.

View all available Meterpreter commands for the current session

```
?
```

Drop from the Meterpreter agent into a native system-level shell to use target-native commands

```
shell
```

- **Meterpreter** functions via **in-memory DLL injection** to avoid writing to disk and maintain stealth.
- **Restricted command sets** in Meterpreter may necessitate dropping to a native shell to access the full suite of system-native binaries.