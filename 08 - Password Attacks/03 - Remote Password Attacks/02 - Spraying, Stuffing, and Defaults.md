# Password Spraying, Stuffing, and Defaults

## Password Spraying

**Password spraying** is used to identify accounts that have not updated a **default or standard password**. Unlike traditional brute-forcing of a single account, this technique attempts a **single password across many different user accounts** to avoid account lockout thresholds.

### Operational Workflow

1. **Identify** a common initialization password (e.g., `ChangeMe123!`) or a default password known to be used by administrators.
2. **Select** the appropriate tool based on the target environment: **Burp Suite** for web applications or **NetExec/Kerbrute** for Active Directory.
3. **Execute** the spray across the identified user list.

|Tool|Service/Environment|Command|
|:--|:--|:--|
|**NetExec**|SMB / Active Directory|`netexec smb <TARGET_IP_RANGE> -u <USER_LIST> -p '<PASSWORD>'`|

## Credential Stuffing

**Credential stuffing** is employed when an attacker has obtained **stolen credentials from a database leak** or a different service. This technique leverages the fact that users frequently **reuse usernames and passwords** across multiple platforms, including enterprise systems.

### Operational Workflow

1. **Acquire** a list of credentials formatted as `username:password`.
2. **Attempt** access to target services (e.g., SSH) using automated tools.

|Tool|Service|Command|
|:--|:--|:--|
|**Hydra**|SSH|`hydra -C <USER_PASS_LIST> ssh://<TARGET_IP>`|

## Default Credentials

Infrastructure components—such as **routers, firewalls, and databases**—often ship with default credentials that administrators may fail to change during setup. These represent a critical risk, particularly in **internal testing environments** where security settings are often overlooked.

### Tooling: Default Credentials Cheat Sheet

This tool automates the process of finding known defaults for specific products or vendors.

**Installation:**

```
pip3 install defaultcreds-cheat-sheet
```

**Usage:**

1. **Search** for the specific product or vendor.
2. **Research** product documentation if tools do not yield results, as documentation often outlines initial setup passwords.
3. **Compile** found credentials into a list for automated testing.

|Action|Command|
|:--|:--|
|**Search for Product Defaults**|`creds search <PRODUCT_NAME>`|

### Common Default Credentials Reference

The following table highlights known default settings for common network hardware found in the sources:

|Vendor/Product|Default Username|Default Password|
|:--|:--|:--|
|**3Com**|`admin`|`Admin`|
|**Belkin**|`admin`|`admin`|
|**BenQ**|`admin`|`Admin`|
|**D-Link**|`admin`|`Admin`|
|**Digicom**|`admin`|`Michelangelo`|
|**Linksys**|`admin`|`admin` / `Admin` / `<blank>`|
|**Linksys**|`root`|`orion99`|
|**Netgear**|`admin`|`password`|

**Attack Implication:** Successfully identifying default credentials can grant full administrative access to critical network infrastructure, facilitating further lateral movement or data exfiltration.