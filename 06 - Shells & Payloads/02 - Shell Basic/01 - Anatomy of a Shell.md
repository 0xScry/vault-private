# Anatomy of a Shell

A shell session consists of a combination of the operating system, a **terminal emulator** application, and a **command language interpreter**. Understanding the specific interpreter in use is critical for determining which commands and scripts are compatible with the target system.

### Terminal Emulators

A terminal emulator is the application used to interact with the shell. While selection on an attack machine is often based on personal preference and workflow, the emulator encountered on a target is typically restricted to what is **natively installed**.

|Terminal Emulator|Operating System|
|:--|:--|
|Windows Terminal|Windows|
|cmder|Windows|
|PuTTY|Windows|
|kitty|Windows, Linux, MacOS|
|Alacritty|Windows, Linux, MacOS|
|xterm|Linux|
|GNOME Terminal|Linux|
|MATE Terminal|Linux|
|Konsole|Linux|
|Terminal|MacOS|
|iTerm2|MacOS|

### Command Language Interpreters

The interpreter processes user instructions and issues tasks to the operating system. These are also referred to as **shell scripting languages** or **Command and Scripting interpreters** within the MITRE ATT&CK Matrix. Terminal emulators are not tied to a single language; for instance, the MATE terminal can run **Bash** or **PowerShell** depending on configuration.

---

### Methodology: Shell Identification

Before executing complex scripts or exploits, you must identify the active command language interpreter to ensure command recognition.

**1. Analyze the Shell Prompt** Observe the visual "clues" in the terminal. A `$` sign typically identifies interpreters such as **Bash**, **Ksh**, or **POSIX**.

**2. Verify via Process Listing** View the active processes to see which interpreter binary is currently executing the session.

**3. Inspect Environment Variables** Check the system's environment variables to see the defined `SHELL` path.

---

### Command Reference

|Command|Goal|
|:--|:--|
|`ps`|Lists running processes to identify the active shell binary (e.g., bash).|
|`env`|Displays environment variables, including the `SHELL` variable.|

**Command Examples:**

```
# Check running processes for shell identification
ps
```

```
# View environment variables to find the SHELL definition
env
```