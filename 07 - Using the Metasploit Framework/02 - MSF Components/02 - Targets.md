### Metasploit Module Reconnaissance and Target Selection

Exploit modules use **Targets** to adapt specific attack code to various operating system versions, service packs, and languages. Proper identification and selection of these targets ensure the exploit uses the correct **return addresses** (e.g., `jmp esp` or `pop/pop/ret`) required for successful execution.

#### 1. Module Inspection

Before configuring an exploit, use the `info` command to perform a technical audit of the module.

- **When to use:** Whenever using a new or unfamiliar module.
- **Why it matters:** It reveals **target dependencies**—such as specific software versions (e.g., JRE 1.6.x) or libraries (e.g., `msvcrt`)—that must be present for the exploit's ROP chain to function. Auditing the module also helps identify potential **artifact generation** to maintain a clean environment.

|Command|Purpose|
|:--|:--|
|`info`|Displays module platform, architecture, license, and vulnerability description.|
|`options`|Shows required parameters for the exploit and payload.|

#### 2. Identifying and Setting Targets

The `show targets` command lists the specific environments the module is designed to exploit.

**Operational Workflow:**

1. **Load the exploit module:** You must be within an exploit's context to see its specific targets.
2. **List available targets:** View the OS versions and service packs supported by the module.
3. **Select the target:** Choose the specific index that matches your target reconnaissance data.

|Command|Description|
|:--|:--|
|`show targets`|Lists all vulnerable OS/service pack versions for the active module.|
|`set target <INDEX_NUMBER>`|Manually selects a specific target version.|

**Target Selection Logic:**

- **Automatic Selection (Index 0):** Use when the specific target version is unknown. Metasploit will attempt **service detection** to identify the OS before launching the attack.
- **Manual Selection:** Use when version-level reconnaissance has already confirmed the OS, service pack, or language. This is often necessary when addresses shift due to **language packs** or software hooks.

#### 3. Configuration Parameters

Exploits require specific network and authentication parameters to establish a connection.

|Parameter|Description|
|:--|:--|
|`RHOSTS`|The address of the victim machine.|
|`RPORT`|The target service port (e.g., 445 for SMB).|
|`LHOST`|The address of your attack machine for reverse connections.|
|`LPORT`|The port your attack machine will listen on.|
|`SRVHOST`|The local host to listen on for browser-based exploits.|
|`URIPATH`|The URI used to trigger the exploit (often randomized).|

#### 4. Critical Dependencies and Failure Conditions

Failure to match the environment to the module's requirements will result in exploit failure.

|Dependency Type|Attack Implication|
|:--|:--|
|**Software Versions**|Some exploits require specific versions of third-party software (e.g., JRE or IE versions) to be installed for memory corruption to work.|
|**Return Addresses**|If the wrong target is selected, the return address (like `jmp esp`) may point to invalid memory, causing the exploit to fail or the service to crash.|
|**Language Packs**|Different languages can shift memory addresses; ensure the target selection accounts for the target's localized OS version.|

**Command Execution Example:**

```
# Example: Configuring a browser-based exploit
msf6 exploit(windows/browser/ie_execcommand_uaf) > info
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target <INDEX_NUMBER>
msf6 exploit(windows/browser/ie_execcommand_uaf) > set RHOSTS <TARGET_IP>
msf6 exploit(windows/browser/ie_execcommand_uaf) > set LHOST <ATTACK_IP>
```