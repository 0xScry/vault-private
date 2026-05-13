## Methodology

1. Identify a controlled user or group SID to serve as the search pivot.
2. If tools are unrestricted, prioritize **BloodHound** for transitive path visualization.
3. If limited to CLI, use **PowerView** with targeted SID filtering and GUID resolution.
4. If PowerView is blocked or unavailable, fall back to native **PowerShell** cmdlets (`Get-Acl`, `Get-ADUser`).
5. Analyze results for high-value ACEs: **User-Force-Change-Password**, **GenericWrite**, **GenericAll**, or **DS-Replication-Get-Changes**.
6. Check for **nested group membership** to identify inherited rights.

---

## ACL Enumeration with PowerView

### Targeted ACL Search

Used when you have control over a specific account and need to identify what objects that account can manipulate. Targeted searches prevent data overload from broad enumeration.

Get SID for a controlled user:

```
$sid = Convert-NameToSid <USERNAME>
```

Identify all objects the user has rights over with human-readable output:

```
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

### Manual GUID Mapping

Used if `-ResolveGUIDs` fails or if you need to manually verify an **ObjectAceType** GUID against the schema.

Map a specific GUID to its display name:

```
$guid = "<GUID>"
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,DistinguishedName,rightsGuid | ? {$_.rightsGuid -eq $guid} | fl
```

- **Tool comparison**
    
    - PowerView (`Get-DomainObjectACL -ResolveGUIDs`) -> Automated resolution -> Prefer for speed and readability.
    - ActiveDirectory Module (`Get-ADObject`) -> Manual schema query -> Prefer when PowerView is unavailable or GUIDs remain unresolved.
- **Gotchas**
    
    - **Module Conflict**: Running the manual `Get-ADObject` mapping cmdlet may cause errors if the PowerView module is already imported in the current session; a **new PowerShell session** may be required.
    - **Performance**: Running `-Identity *` in large environments will cause **significant delays**, often taking several minutes to return results.

## Native PowerShell ACL Discovery

### Living off the Land Enumeration

Used when third-party tools like PowerView or SharpHound are restricted by the client or blocked by security controls.

Generate a user list for the loop:

```
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > <FILE_PATH>
```

Loop through users to find access rights for a specific identity:

```
foreach($line in [System.IO.File]::ReadLines("<FILE_PATH>")) {get-acl "AD:$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match '<DOMAIN>\<USERNAME>'}}
```

- **Tool comparison**
    
    - PowerView -> Optimized LDAP queries -> Prefer for speed and efficiency.
    - Native `Get-Acl` -> Standard .NET/PowerShell methods -> Prefer for **stealth/bypass** when custom binaries/scripts are forbidden.
- **Gotchas**
    
    - **Efficiency**: The native `foreach` loop is **extremely slow** in large environments compared to tool-based enumeration.

## Group Nesting and Transitive Control

### Inherited Rights Analysis

Used when a user has no direct interesting rights but is a member of a group that may be nested within more powerful groups.

Check group nesting to find inherited permissions:

```
Get-DomainGroup -Identity "<GROUP_NAME>" | select memberof
```

- **Dangerous / misconfigured settings**
    - **GenericWrite**: Allows adding any user (including yourself) to a group to inherit its rights.
    - **GenericAll**: Full control over an object, including password resets and attribute manipulation.
    - **DS-Replication-Get-Changes**: When combined with **DS-Replication-Get-Changes-In-Filtered-Set**, allows performing a **DCSync attack**.

## Graphical Path Analysis

### BloodHound Path Discovery

Used for identifying complex, multi-hop attack paths that are difficult to track manually through CLI output.

- **Tool comparison**
    
    - PowerView -> Manual tracking of each hop -> Prefer for surgical verification of specific ACEs.
    - BloodHound -> Automated transitive pathing -> Prefer for **time-boxed assessments** to find the shortest path to Domain Admin.
- **Edge cases**
    
    - **Transitive Object Control**: Use the "Node Info" tab to see the total count of objects reachable via complex ACL chains.
    - **Pre-built Queries**: Use built-in queries to quickly find all users with **DCSync** rights across the domain.