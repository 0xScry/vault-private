1. Verify the plugin exists in the framework directory.
2. If absent, copy the desired `.rb` file into the local plugin path using elevated privileges.
3. Launch `msfconsole` and initialize the plugin to expose new command sets to the API.
4. Call the plugin-specific help menu to identify available interactions.

---

## Plugin Management

**When to use** — Need to integrate 3rd party software (Nessus, OpenVAS, SQLMap) directly into `msfconsole` to automate documentation and data import/export into the current database.

Check available plugins in the default installation directory

```
ls /usr/share/metasploit-framework/plugins
```

Initialize a plugin within the framework

```
load <SERVICE_NAME>
```

Verify successful loading and view extended command set

```
<SERVICE_NAME>_help
```

**Gotchas** **Failed to load plugin** occurs if the `.rb` file is missing from the directory or the path provided is incorrect.

## Custom Plugin Installation

**When to use** — Required functionality or specific tradecraft scripts (e.g., `pentest.rb`) are missing from the default Parrot OS or Metasploit updates.

Copy a new Ruby plugin file to the framework plugin directory

```
sudo cp <FILE_PATH>.rb /usr/share/metasploit-framework/plugins/<FILE_PATH>.rb
```

Load the custom plugin to extend the main help menu with new command categories

```
load <FILE_PATH>
```

**Gotchas** **Permissions errors** will prevent the framework from accessing newly copied `.rb` files in the restricted `/usr/share/` path.

## Framework Mixins

**When to use** — Developing custom Ruby scripts where you need to include functionality from other classes without using formal inheritance.

Include a module within a script to provide specific methods

```
include <MODULE_NAME>
```

**Edge cases** Mixins are primarily for developers; for standard assessments, **focus on plugins** for framework manipulation.