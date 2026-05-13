1. Map domain object relationships and identify edges using BloodHound or PowerView.
2. Filter for high-impact ACEs: WriteDacl, WriteOwner, GenericWrite, or AddMember.
3. Check ACE order; verify no **access denied ACE** exists above the target allowed entry.
4. If rights allow password resets or group modifications, verify if the attack is **destructive** and confirm client approval.
5. Execute modification to achieve lateral or vertical movement.
6. Revert all AD changes and document original vs. modified states for the report.

---

## ACL Enumeration and Visualization

Standard AD misconfigurations (low-hanging fruit) are remediated, requiring path analysis for lateral or vertical movement.

Enumerate and visualize exploitable edges

```
BloodHound
```

Enumerate ACEs and extended rights (e.g., Unexpire-Password, Reanimate-Tombstones)

```
PowerView
```

- PowerView -> Use for CLI-based enumeration of specific object permissions and extended rights.
- BloodHound -> Use for visual path analysis and identifying complex chains of trust.
- ADUC (Advanced Security Settings) -> Use for manual verification of DACL/SACL entries and auditing.

> ⚠️ Gap: Source identifies tools (PowerView, BloodHound) but does not provide specific command-line syntax or flags for execution.

- **access denied ACE**: Found during top-to-bottom list evaluation, it terminates the check and prevents access regardless of subsequent allowed entries.

## WriteDacl and WriteOwner Abuse

Object control allows granting new rights or taking ownership to bypass existing restrictions.

Leverage WriteDacl/WriteOwner to grant rights or add members

```
PowerView
```

- WriteDacl: Allows the principal to modify the DACL and grant themselves or others specific rights like GenericWrite.
- WriteOwner: Allows the principal to take ownership of the object, subsequently allowing DACL modification.

## Group Membership Management Abuse

Principal has permissions to add or remove users from specific groups, potentially targeting privileged or built-in AD groups.

Add a controlled account to a privileged group

```
PowerView
```

- Help Desk users or IT staff with delegated rights over specific OUs or groups.
    
- Software-specific ACL changes (e.g., Exchange installation) that introduce excessive rights.
    
- **destructive modification**: Adding or removing users from groups can disrupt production workflows; changes must be reverted and documented.
    

## GMSA Password Retrieval

Controlled principal has the ReadGMSAPassword right over a Group Managed Service Account.

Read the password for a Group Managed Service Account

```
GMSAPasswordReader
```

- ReadGMSAPassword edge appearing in BloodHound data.
    
- **incomplete enumeration**: Failing to check for less common extended rights like Reanimate-Tombstones may result in missing viable persistence or movement vectors.