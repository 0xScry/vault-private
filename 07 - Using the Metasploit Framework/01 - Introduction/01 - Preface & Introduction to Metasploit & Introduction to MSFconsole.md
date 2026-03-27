# Metasploit Framework Fundamentals

The **Metasploit Project** is a Ruby-based, modular platform used to write, test, and execute exploit code. It functions as a "Swiss army knife" for penetration testing, providing tools to **enumerate networks**, **execute attacks**, and **evade detection**.

## Metasploit Versions

Metasploit is split into two primary versions based on use case:

|Version|Use Case|Key Features|
|:--|:--|:--|
|**Metasploit Framework**|Open Source / Community|CLI-based, core manual exploitation tools, modular database|
|**Metasploit Pro**|Commercial / Enterprise|GUI, automation (Task Chains), Vulnerability Validation, Social Engineering, VPN Pivoting|

---

## Operational Workflow & Methodology

A structured engagement with Metasploit follows a five-step methodology to ensure comprehensive coverage and avoid "tunnel vision".

1. **Enumeration**: Perform a thorough scan of the `<TARGET_IP>` to identify running services and, crucially, their **versions**. Versions are key to determining if a target is vulnerable.
2. **Preparation**: Involves **vulnerability research** and **code auditing** to select the appropriate exploit module.
3. **Exploitation**: The execution of the chosen module against the target.
4. **Privilege Escalation**: Moving from a low-privilege foothold to higher system access.
5. **Post-Exploitation**: Activities including **pivoting** to internal networks and **data exfiltration**.

---

## Initializing and Updating MSF

### Updating the Framework

Ensure the framework and its module database are current to include the latest public exploits. Modern distributions handle this via the package manager.

|Goal|Command|
|:--|:--|
|Update MSF & Modules|`sudo apt update && sudo apt install metasploit-framework`|

### Launching msfconsole

The `msfconsole` is the centralized interface for the framework.

|Option|Command|Why/When to Use|
|:--|:--|:--|
|Standard Launch|`msfconsole`|General use; displays version stats and splash art|
|Quiet Mode|`msfconsole -q`|Use to suppress the banner and start directly at the prompt|
|Display Help|`help`|Use within `msfconsole` to view all available commands|

---

## File System & Architecture

Understanding the directory structure is essential for importing new modules or creating custom ones. Base files are located at `/usr/share/metasploit-framework`.

### Core Directories

|Directory|Content Description|
|:--|:--|
|`data` / `lib`|Functioning parts of the `msfconsole` interface|
|`documentation`|Technical details regarding the project|
|`modules`|The core exploit, auxiliary, and payload files|
|`plugins`|Add-ons for automation and extra functionality (e.g., `sqlmap`, `nessus`)|
|`scripts`|Meterpreter and shell functionality|
|`tools`|Command-line utilities called from the console menu|

### Module Categories

Modules are stored in `/usr/share/metasploit-framework/modules` and categorized by function:

- **auxiliary**: Scanners, crawlers, and fuzzers.
- **encoders**: Used to obfuscate payloads.
- **evasion**: Tools to bypass security controls.
- **exploits**: Code that takes advantage of a vulnerability.
- **nops**: No-operation instructions for payload padding.
- **payloads**: Code that runs on the target after successful exploitation (e.g., reverse shells).
- **post**: Modules for post-exploitation tasks.

---

## Professional Discipline & Tool Usage

- **Avoid Tunnel Vision**: Do not rely on automated tools as the sole backbone of an assessment; they are time-savers, not a replacement for manual research.
- **Technical Literacy**: Read technical documentation for every tool to avoid unpredictable behaviors that could leave traces on a target or expose the attack platform.
- **Efficiency**: Use Metasploit to handle high-impact, high-turnover issues first, as time for a "complete" assessment is rarely available.