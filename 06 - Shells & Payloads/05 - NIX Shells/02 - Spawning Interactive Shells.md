# Spawning Interactive Shells

### Operational Context

Initial shell sessions are often limited (referred to as **jail shells**), restricting available commands and prompt functionality. While Python is a common tool for upgrading to a **TTY Bourne shell**, it may not be installed on all targets. In such cases, alternative interpreters or system utilities must be used to spawn an interactive environment.

### Shell Upgrade Methodology

1. **Identify Available Interpreters**: Determine if the system has native shells (`/bin/sh`, `/bin/bash`) or programming languages (Perl, Ruby, Lua, AWK) installed.
2. **Execute Spawn Command**: Use the specific syntax for the available language or utility to invoke `/bin/sh` or `/bin/bash` in interactive mode.
3. **Validate Stability**: Once upgraded, verify if the shell supports job control or complex commands like `sudo -l`.

### Command Reference: Language-Based Shells

|Language|Command|Notes|
|:--|:--|:--|
|**Bourne Shell**|`/bin/sh -i`|Executes the shell in interactive mode (`-i`).|
|**Perl**|`perl -e 'exec "/bin/sh";'`|Should be run from a script.|
|**Ruby**|`ruby: exec "/bin/sh"`|Should be run from a script.|
|**Lua**|`lua: os.execute('/bin/sh')`|Uses the `os.execute` method; should be run from a script.|
|**AWK**|`awk 'BEGIN {system("/bin/sh")}'`|Uses AWK's C-like pattern processing to trigger a system call.|

### Command Reference: Utility-Based Shells

|Utility|Command|Scenario / Context|
|:--|:--|:--|
|**Find**|`find . -exec /bin/sh \; -quit`|Uses the `-exec` option to initiate the shell directly.|
|**Find (via AWK)**|`find / -name <FILENAME> -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;`|**Edge Case**: Searches for a specific file first. If the file is not found, no shell is attained.|
|**VIM**|`vim -c ':!/bin/sh'`|**Niche Situation**: Escapes the VIM text editor to a shell session via command-line flags.|
|**VIM (Internal)**|`:set shell=/bin/sh` followed by `:shell`|Used if already inside an active VIM session.|

---

### Post-Exploitation: Permissions & Enumeration

Once an interactive shell is established, the next priority is determining account permissions to identify **privilege escalation vectors**.

|Goal|Command|Decision Impact|
|:--|:--|:--|
|**Check File Permissions**|`ls -la <PATH_TO_FILE>`|Identifies properties and owner permissions over specific binaries.|
|**Check Sudo Privileges**|`sudo -l`|**Requirement**: Requires a **stable interactive shell**. Unstable shells may return nothing.|

**Attack Implications**:

- Establishing a stable shell unlocks the ability to run `sudo -l`, which may reveal **NOPASSWD** configurations or other misconfigured binaries that allow for immediate privilege escalation.