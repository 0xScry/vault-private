### **Metasploit: Manual Module Import and Development**

#### **Overview**

While a full framework update ensures the latest modules are present, **manual importing** is preferred when a specific exploit is required without performing a full upgrade or when using modules not yet pushed to the official Metasploit-framework branch.

---

#### **1. Finding and Filtering Modules**

When a module is missing from the local `msfconsole` search results, use external databases or CLI tools to locate the specific Ruby (`.rb`) script.

- **ExploitDB:** Use the **Metasploit Framework (MSF)** tag to filter for compatible scripts.
- **Searchsploit:** Use the CLI to find local copies of ExploitDB entries.

|Command|Description|
|:--|:--|
|`searchsploit <TERM>`|Search for exploits by name or service.|
|`searchsploit -t <TERM> --exclude=".py"`|Filter results to show only potential Metasploit-compatible Ruby scripts.|

**Decision Point:** Not all `.rb` files are Metasploit modules; some are standalone Ruby scripts. Verify the file contains Metasploit-specific **mixins** and class structures before attempting to import.

---

#### **2. Manual Installation Workflow**

Metasploit requires a specific directory structure to recognize new modules. You must mirror the official framework structure in your local directory.

**Operational Steps:**

1. **Identify the target directory:** The framework resides in `/usr/share/metasploit-framework/modules/` or the hidden user directory `~/.msf4/modules/`.
2. **Create sub-directories:** If the specific category folders (e.g., `exploits/unix/webapp/`) do not exist in `~/.msf4/`, they must be created manually to match the framework's hierarchy.
3. **Apply naming conventions:** Modules must use **snake_case** (alphanumeric and underscores). Improper naming prevents `msfconsole` from recognizing the file.
4. **Copy the module:**
    
    ```
    cp <DOWNLOADED_SCRIPT>.rb /usr/share/metasploit-framework/modules/exploits/<PATH>/<MODULE_NAME>.rb
    ```
    

---

#### **3. Loading and Initializing Modules**

Once the file is placed, it must be loaded into the current session to be searchable and usable.

|Command|Goal|
|:--|:--|
|`msfconsole -m /usr/share/metasploit-framework/modules/`|Launch Metasploit while explicitly loading a specific module directory.|
|`loadpath /usr/share/metasploit-framework/modules/`|Load new modules from within an active `msfconsole` session.|
|`reload_all`|Refresh the framework database to include newly added modules.|
|`use exploit/<PATH>/<MODULE_NAME>`|Select the newly installed module for use.|

---

#### **4. Porting and Writing Custom Modules**

When a script (Python, PHP, etc.) needs to be adapted for Metasploit, it is common practice to **repurpose existing modules** as boilerplate code rather than writing from scratch.

**Key Component: Mixins** Mixins provide the methods required for the module to interact with the target. Including only necessary mixins keeps the module efficient.

|Mixin|Function|
|:--|:--|
|`Msf::Exploit::Remote::HttpClient`|Methods for acting as an HTTP client against a server.|
|`Msf::Exploit::PhpEXE`|Generates first-stage PHP payloads.|
|`Msf::Exploit::FileDropper`|Handles file transfer and post-session cleanup.|
|`Msf::Auxiliary::Report`|Methods for reporting data back to the MSF database.|

**Module Configuration (The `initialize` Block):** Modify the metadata to match the new exploit, including:

- **Name/Description:** Clear identification of the vulnerability.
- **References:** CVE numbers and discovery URLs.
- **Options:** Define required variables using `register_options` (e.g., `TARGETURI`, `USERNAME`, `PASSWORD`).
- **Payload Info:** Set the `Platform` and `Arch` (e.g., `php`, `ARCH_PHP`).

**Operational Command Reference:**

```
# Example of using a module and viewing its required parameters
msf6 > use exploit/<PATH>/<MODULE_NAME>
msf6 exploit(...) > show options
```

**Variables Table:**

| Parameter | Description                                      |
| :-------- | :----------------------------------------------- |
| `RHOSTS`  | `<TARGET_IP>` or CIDR range.                     |
| `RPORT`   | `<PORT>` (default is often 80 for web exploits). |
| `USER`    | `<USERNAME>` for authentication.                 |
| `PASS`    | `<PASSWORD>` for authentication.                 |