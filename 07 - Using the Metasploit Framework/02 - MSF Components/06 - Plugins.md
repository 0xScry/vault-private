# Metasploit Framework: Plugins and Mixins

## Metasploit Plugins

**Plugins** are third-party software integrated into the framework to **automate repetitive tasks**, **add new commands**, and **extend functionality**.

### Why Use Plugins

- **Centralized Documentation**: Plugins automatically document results into the active database, making hosts, services, and vulnerabilities available at a glance.
- **Workflow Efficiency**: They eliminate the need to manually cycle between different software to import/export results or re-enter parameters.
- **Framework Manipulation**: Plugins work directly with the API, allowing for the manipulation of the entire framework.

### Managing Plugins

#### 1. List Available Plugins

Check the default installation directory to identify available plugins.

```
ls /usr/share/metasploit-framework/plugins
```

#### 2. Load a Plugin

Initialize the plugin within the `msfconsole`. A successful load displays a greeting and specific usage instructions.

```
load <PLUGIN_NAME>
```

#### 3. Access Plugin Help

Plugins often extend the help menu or provide a specific help command to list available interactions.

```
<PLUGIN_NAME>_help
```

OR

```
help
```

### Command Reference: Nessus Plugin

|Command|Help Text|
|:--|:--|
|`nessus_connect`|Connect to a Nessus server|
|`nessus_logout`|Logout from the Nessus server|
|`nessus_login`|Login into the connected Nessus server with a different username|
|`nessus_user_del`|Delete a Nessus User|
|`nessus_user_passwd`|Change Nessus Users Password|
|`nessus_policy_list`|List all policies|
|`nessus_policy_del`|Delete a policy|

### Installing Custom Plugins

While popular plugins are updated via the OS distribution (e.g., Parrot OS), custom plugins must be installed manually.

**Workflow:**

1. **Download** the plugin `.rb` file from the developer's page.
2. **Move** the file to the Metasploit plugins directory: `/usr/share/metasploit-framework/plugins/`.
3. **Ensure** the file has proper permissions.
4. **Load** the plugin in `msfconsole` to verify installation.

**Operational Example (Installing `pentest.rb`):**

```
# Clone the repository containing the plugin
git clone <PLUGIN_REPO_URL>

# Copy the plugin file to the MSF directory
sudo cp ./<REPO_NAME>/<PLUGIN_FILE>.rb /usr/share/metasploit-framework/plugins/<PLUGIN_FILE>.rb

# Launch MSF and load the plugin
msfconsole -q
msf6 > load <PLUGIN_FILE>
```

### Failure Conditions

If a plugin is not installed correctly or the file path is incorrect, `msfconsole` will return a **"Failed to load plugin"** error indicating it cannot find the file.

---

## Metasploit Mixins

**Mixins** are a feature of the Ruby language that provide **flexibility** to both script creators and users within the framework.

### Technical Concept

- **Inclusion vs. Inheritance**: Mixins are classes that act as methods for use by other classes without being a parent class. This is defined as **inclusion** rather than inheritance.
- **Implementation**: They are implemented using the `include` keyword followed by the module name.
- **Decision Logic**: While not critical for basic assessments, Mixins are essential for users looking to perform **complex customization** of Metasploit.

### Pre-installed Tooling and Mixins

|Category|Examples|
|:--|:--|
|**Scanning/Assessment**|nMap, NexPose, Nessus|
|**Post-Exploitation**|Mimikatz (V.1), Stdapi, Incognito|
|**Advanced Extensions**|Railgun Priv, Darkoperator's Mixins|