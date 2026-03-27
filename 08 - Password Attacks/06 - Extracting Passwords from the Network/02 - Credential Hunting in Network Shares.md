# **Credential Hunting in Network Shares**

Corporate network shares are essential for collaboration but frequently contain **plaintext credentials** or **configuration files** left behind by employees. Attackers hunt these shares to uncover hidden secrets and escalate privileges.

### **Initial Manual Discovery (Windows)**

Before using automated tools, perform basic command-line searches to identify sensitive file patterns.

|Command|Purpose|
|:--|:--|
|`Get-ChildItem -Recurse -Include *. \<TARGET_IP><SHARE_NAME> \|Select-String -Pattern `|

---

### **Automated Hunting from Windows**

#### **1. Snaffler**

**Snaffler** is a C# tool designed to run on a **domain-joined machine**. It automatically identifies accessible shares and searches for "interesting" files that may contain credentials.

**Operational Workflow:**

1. Execute a basic search to let the tool discover DFS paths and computers from Active Directory.
2. Review the output for **red-coded** findings, which indicate high-probability credential matches (e.g., `unattend.xml` files containing administrator passwords).

**Command Reference:**

|Command|Description|
|:--|:--|
|`Snaffler.exe -s`|Executes a basic automated search across the domain.|

**Attack Implications:**

- Automatically discovers **DFS paths** and all computers in the domain.
- Identifies readable shares and flags sensitive data like `AdministratorPassword` tags in configuration files.

#### **2. PowerHuntShares**

**PowerHuntShares** is a PowerShell script that provides a comprehensive audit of SMB shares. It is ideal when a **visual report** is needed for analysis or when the attack machine is **not domain-joined**.

**Scenario Context:**

- Use when manual review of large data sets is required via an **HTML report**.
- **Warning:** This tool can take **hours to run** in large environments.

**Command Reference:**

|Command|Parameter|Description|
|:--|:--|:--|
|`Invoke-HuntSMBShares -Threads <NUMBER> -OutputDirectory <PATH>`|`-Threads`|Sets the speed of enumeration.|
||`-OutputDirectory`|Location for the HTML summary and CSV files.|

**Capabilities:**

- Checks for **TCP 445** availability.
- Identifies shares with **excessive privileges** or high-risk directory listings.
- Generates **timelines** for last written and last accessed files.

---

### **Automated Hunting from Linux**

#### **1. MANSPIDER**

**MANSPIDER** is used to scan SMB shares remotely from a Linux host. It is best executed via **Docker** to ensure all dependencies are met.

**Scenario Context:**

- Use when you do not have access to a domain-joined Windows host or prefer remote scanning.

**Command Reference:**

|Command|Description|
|:--|:--|
|`docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider <TARGET_IP> -c '<PATTERN>' -u '<USERNAME>' -p '<PASSWORD>'`|Scans the target IP for file content matching the specified pattern using provided credentials.|

#### **2. NetExec (NXC)**

**NetExec** includes a **spidering** function to search through shares for specific content.

**Command Reference:**

|Command|Description|
|:--|:--|
|`nxc smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' --spider <SHARE_NAME> --content --pattern "<PATTERN>"`|Spiders a specific share on the target to find files containing the defined string pattern.|

---

### **Risk Assessment: High-Risk Share Configurations**

|Configuration Issue|Impact|
|:--|:--|
|**Excessive Privileges**|Allows unauthorized users to read or write sensitive configuration/credential files.|
|**Unprotected `unattend.xml`**|Often contains plaintext or obfuscated `AdministratorPassword` values.|
|**Sensitive File Formats**|Files like `.wim` (deployment images) or configuration files often reside in `ADMIN$` or `C$` shares and may leak secrets.|

**Methodology Note:** Automated tools generate significant output and frequent **false positives**; manual review of "loot" directories and HTML reports is always required to verify findings.