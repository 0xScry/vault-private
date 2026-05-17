
## External Module Discovery and Import

Target exploit is verified on ExploitDB but missing from local `search` results, requiring manual installation into the framework directory structure,.

Search ExploitDB via CLI for Ruby scripts specifically tagged for Metasploit

```
searchsploit <SERVICE_NAME> -t --exclude=".py"
```

**Tool comparison**

- **searchsploit**
    
    - `searchsploit <TERM>`
    - Prefer for local CLI-based offline discovery.
- **ExploitDB (Web)**
    
    - Filter by **Metasploit Framework (MSF)** tag.
    - Prefer for visual filtering by author, port, or platform.
- **snake-case** naming: **Alphanumeric** characters and **underscores** only; **dashes** will cause **module recognition failure**.
    
- Directory structure: Ensure the local path in `~/.msf4/modules/` mirrors the official `/usr/share/metasploit-framework/modules/` hierarchy (e.g., `exploits/<PLATFORM>/<SERVICE>/`) or the module will not be found.
    

> ⚠️ Gap: Manual module imports into `/usr/share/metasploit-framework/` usually require **root privileges**, which may lead to **permission denied** errors if using a standard user account.

---

## Custom Module Loading

New `.rb` module has been moved to the local filesystem and needs to be registered within the active MSF instance without a full framework upgrade,.

Launch MSF and automatically include a custom module directory

```
msfconsole -m /usr/share/metasploit-framework/modules/
```

Load modules from a specific path while the console is already running

```
loadpath /usr/share/metasploit-framework/modules/
```

Force MSF to refresh the module database to pick up newly added files

```
reload_all
```

**Gotchas**

- **Ruby script compatibility**: Not all `.rb` files are MSF modules; scripts lacking **Metasploit-compatible code** will fail to load.

---

## Porting Standalone Exploits to MSF

Converting a Python or PHP PoC into a Ruby module by utilizing existing framework mixins and boilerplate code,.

Identify an existing module to use as a functional template

```
ls /usr/share/metasploit-framework/modules/exploits/<PLATFORM>/<PROTOCOL>/ | grep <SERVICE_NAME>
```

**Dangerous / misconfigured settings**

- Indentation: MSF modules **must use hard tabs**; spaces may cause **syntax or loading errors**.
- Unnecessary Mixins: Including unused mixins like `Msf::Exploit::FileDropper` adds overhead; remove them if the exploit doesn't handle file cleanup,.

**Edge cases**

- **Custom Network Environments**: Proprietary code or non-standard protocols may require writing a module from scratch using the `Msf::Exploit::Remote` class if no boilerplate exists.

**Gotchas**

- **Validation failure**: Ensure `register_options` includes all required variables (e.g., `TARGETURI`, `USER`, `PASSWORDS`) or the module will fail to execute,.