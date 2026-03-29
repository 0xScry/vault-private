# Infiltrating Unix/Linux Systems

Establishing a shell session on a Unix/Linux system is a critical step for **pivoting** within internal networks, as over 70% of web servers run on Unix-based OSs.

## Phase 1: Initial Enumeration

The goal is to identify the **web stack**, **operating system**, and any **hosted applications** that may contain vulnerabilities.

1. **Perform a service scan** to identify open ports and versions.
2. **Analyze the output** for version numbers (e.g., Apache, PHP) and the Linux distribution (e.g., CentOS).
3. **Navigate to the web application** via a browser using the `<TARGET_IP>` to identify the specific software in use.

### Command Reference: Enumeration

|Command|Description|
|:--|:--|
|`nmap -sC -sV <TARGET_IP>`|Performs a script scan and version detection on the target.|

---

## Phase 2: Vulnerability Discovery

Once an application is identified (e.g., **rConfig**), you must determine its version to find matching exploits. Tools like rConfig are high-value targets because they manage network appliances; compromising them can lead to control over the entire network.

1. **Locate the version number**, often found at the bottom of login pages.
2. **Search for public exploits** (CVEs, PoCs) using search engines or specialized databases like Exploit-DB.
3. **Search Metasploit** for built-in modules.

### Command Reference: Vulnerability Searching

|Command|Description|
|:--|:--|
|`msf6 > search rconfig`|Searches the Metasploit Framework for modules related to rConfig.|

### Handling Missing Metasploit Modules

If a known exploit (e.g., from a GitHub repo) is missing from your local Metasploit instance, you can manually add it.

|Step|Action|
|:--|:--|
|1|Locate the local Metasploit exploit directory using `locate exploits`.|
|2|Save the `.rb` exploit file into the appropriate subdirectory (e.g., `/usr/share/metasploit-framework/modules/exploits/linux/http/`).|
|3|Update the framework using `apt update; apt install metasploit-framework`.|

---

## Phase 3: Exploitation (Metasploit)

Use Metasploit to deliver a payload and establish a session once a vulnerable application and matching module are confirmed.

1. **Select the module** and configure the required options (RHOSTS, LHOST, etc.).
2. **Launch the exploit** to trigger the payload and open a session.

### Command Reference: MSF Execution

|Command|Description|
|:--|:--|
|`use exploit/linux/http/<MODULE_NAME>`|Selects the specific exploit module.|
|`exploit`|Executes the exploit against the target.|

---

## Phase 4: Shell Stabilization

Payloads executed by web services (like Apache) often result in a **non-TTY shell**. These shells lack a command prompt and do not support interactive commands like `su` or `sudo`, which are necessary for **privilege escalation**.

### Spawning a TTY Shell with Python

If Python is installed, use it to manually spawn a TTY shell to gain access to a prompt and more system commands.

|Step|Goal|Command|
|:--|:--|:--|
|1|Verify Python exists|`which python`|
|2|Drop into system shell|`shell` (within Meterpreter)|
|3|**Spawn TTY**|`python -c 'import pty; pty.spawn("/bin/sh")'`|

### Attack Implications

- **Initial Access:** Typically as a low-privileged user (e.g., `apache`).
- **Stabilization:** Spawning a TTY shell unlocks the ability to use interactive tools for further movement.
- **Critical Discovery:** Exploiting management tools like rConfig can provide **admin access** to network appliances.