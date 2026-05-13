#1. Check **RealTimeProtectionEnabled** status to determine if tools like PowerView will be blocked immediately upon execution.
2. Identify the current PowerShell **LanguageMode** to see if COM objects or .NET types are restricted.
3. Audit AppLocker **RuleCollections** to find **Deny** actions on standard paths and look for missed executable locations.
4. Enumerate LAPS permissions to identify users with **All Extended Rights** or delegated read access to local admin passwords.
5. Retrieve cleartext passwords for LAPS-enabled hosts where the current context has read permissions.

---

## Windows Defender Enumeration

Check for active host-based protections after gaining a foothold to avoid tool detection.

Check status of all Defender features

```
Get-MpComputerStatus
```

- **RealTimeProtectionEnabled** set to **True** will block common tools like PowerView.

## PowerShell Language Mode Verification

Identify environment lockdowns that prevent the use of COM objects, XAML-based workflows, or PowerShell classes.

Query the current session language mode

```
$ExecutionContext.SessionState.LanguageMode
```

- **ConstrainedLanguage** mode is active, many advanced offensive PowerShell features and scripts will fail to execute.

## AppLocker Policy Discovery

Identify application whitelisting restrictions when standard binaries like `powershell.exe` are blocked or limited to specific directories.

View all effective rules and identify Deny/Allow actions

```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

- **Everyone** group allowed to run applications in `%PROGRAMFILES%` and `%WINDIR%`.
    
- **local Administrators** group allowed to run all applications via the `{*}` path condition.
    
- Organizations often block `%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe` but miss `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` or `PowerShell_ISE.exe`.
    
- **Action: Deny** rules override allow rules and will cause execution to fail even if the user is in a group that otherwise has permission.
    

## LAPS Rights Enumeration

Locate delegated groups and users with rights to read cleartext local administrator passwords.

Parse ExtendedRights for all LAPS-enabled computers to find delegated groups

```
Find-LAPSDelegatedGroups
```

Check for users with "All Extended Rights" who can read passwords but may not be in a delegated group

```
Find-AdmPwdExtendedRights
```

- Accounts that joined a computer to the domain receive **All Extended Rights** over that host, enabling them to read LAPS passwords.

> ⚠️ Gap: The source references these functions as part of `LAPSToolkit`, which is not a native Windows cmdlet and must be imported to the session.

## LAPS Password Recovery

Retrieve cleartext local administrator passwords and expiration dates for targeted hosts where read access is confirmed.

Display passwords for machines where the current user context has read permissions

```
Get-LAPSComputers
```

- **Empty password field** in the output indicates the current user lacks sufficient privileges to read the `ms-Mcs-AdmPwd` attribute for that specific host.