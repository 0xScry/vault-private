# Enumerating Security Controls

**Post-foothold enumeration** of security controls is critical to inform tool selection and avoid detection. Security products can block exploitation/post-exploitation tools (e.g., PowerView) or restrict access to specific commands and directories. Organizations may not apply these controls equally across all hosts, making **per-machine enumeration** necessary to identify weaker targets.

---

### Windows Defender Enumeration

Check the state of Windows Defender to determine if your toolkit will be flagged or blocked by real-time signatures.

|Command|Purpose|Key Parameter|
|:--|:--|:--|
|`Get-MpComputerStatus`|Displays current Defender status and configuration.|`RealTimeProtectionEnabled`|

**Operational Steps:**

1. Execute the command to check the host's defensive state.
2. **Decision Point:** If `RealTimeProtectionEnabled` is **True**, tools like PowerView will likely be blocked, necessitating a bypass or tool modification.

---

### AppLocker & Application Whitelisting

AppLocker controls which executables, scripts, and DLLs are permitted to run. It is often used to block access to `cmd.exe` or `powershell.exe`.

|Command|Purpose|
|:--|:--|
|`Get-AppLockerPolicy -Effective \|select -ExpandProperty RuleCollections`|

**Attack Implications:**

- **Path-Based Bypasses:** Admins often block the default 64-bit PowerShell path but neglect others. If the default is blocked, check for access to:
    - `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe`
    - `PowerShell_ISE.exe`
- **Rule Logic:** Default rules often allow members of the **Everyone** group to run applications in `%PROGRAMFILES%` or `%WINDIR%`, while restricting others.

---

### PowerShell Constrained Language Mode (CLM)

CLM restricts PowerShell features to prevent effective post-exploitation, blocking COM objects, specific .NET types, and XAML-based workflows.

|Command|Purpose|
|:--|:--|
|`$ExecutionContext.SessionState.LanguageMode`|Identifies if the session is in `FullLanguage` or `ConstrainedLanguage` mode.|

**Why it matters:** If the output is `ConstrainedLanguage`, many advanced scripts and automation tools will fail to execute correctly.

---

### Local Administrator Password Solution (LAPS)

LAPS randomizes local administrator passwords. Enumerating LAPS helps identify which accounts can read these passwords and which machines lack protection.

|Command|Purpose|
|:--|:--|
|`Find-LAPSDelegatedGroups`|Finds groups delegated with rights to read LAPS passwords.|
|`Find-AdmPwdExtendedRights`|Identifies users with "All Extended Rights" (can read passwords) over computers.|
|`Get-LAPSComputers`|Lists LAPS-enabled computers, password expiration, and cleartext passwords (if authorized).|

**Scenario Context:**

- **Identifying Targets:** Use `Find-LAPSDelegatedGroups` to find users in protected groups or OUs who have password read access.
- **Misconfiguration Check:** Accounts that join a computer to the domain receive **All Extended Rights** by default, allowing them to read LAPS passwords.
- **Credential Harvest:** If your current user has sufficient rights, `Get-LAPSComputers` can provide cleartext local admin passwords for lateral movement.

#### Dangerous Delegations

|Identity|Reason|Impact|
|:--|:--|:--|
|Non-Admin User with `All Extended Rights`|Misconfigured Permission|Can read randomized local admin passwords for targeted hosts.|