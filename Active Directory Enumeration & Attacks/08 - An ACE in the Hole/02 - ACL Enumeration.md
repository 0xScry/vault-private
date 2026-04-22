# Active Directory ACL Enumeration

### Methodology Overview

ACL enumeration is used to identify **Access Control Entries (ACEs)** that grant specific permissions over domain objects. While broad enumeration (e.g., `Find-InterestingDomainAcl`) provides a complete list of misconfigurations, the volume of data is often too large for time-boxed assessments.

**Targeted enumeration**—starting from a compromised user account—is the preferred methodology to identify actionable attack paths quickly.

---

### Targeted Enumeration with PowerView

Use this technique to determine exactly what objects a compromised user can manipulate. PowerView is preferred for its ability to filter by **Security Identifier (SID)** and resolve raw GUIDs into human-readable rights.

**Operational Workflow:**

1. **Identify User SID:** Convert the username to a SID to enable precise filtering.
2. **Enumerate ACEs:** Search for all domain objects where the controlled SID has permissions.
3. **Resolve GUIDs:** Use the `-ResolveGUIDs` flag. Without it, the `ObjectAceType` returns a raw GUID, making the specific right (e.g., "Reset Password") unreadable.

| Command                                                                              | Purpose                                     |
| :----------------------------------------------------------------------------------- | :------------------------------------------ |
| `Import-Module .\PowerView.ps1`                                                      | Loads the PowerView module.                 |
| `$sid = Convert-NameToSid <USERNAME>`                                                | Converts a username to a SID for filtering. |
| `Get-DomainObjectACL -ResolveGUIDs -Identity * \|? {$_.SecurityIdentifier -eq $sid}` |                                             |

---

### Manual GUID Mapping (Reverse Search)

Use this technique if PowerView fails to resolve a GUID or if you are using native tools that only return raw `ObjectAceType` values.

**Scenario Context:** If PowerView has already been imported in the session, the native command below may result in an error; use a new PowerShell session if needed.

|Command|Purpose|
|:--|:--|
|`$guid= "<GUID>"`|Sets the target GUID variable.|
|`Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * \|Select Name,DisplayName,DistinguishedName,rightsGuid \|

---

### Living-off-the-Land (Native Cmdlets)

Use this method when restricted to native tools and unable to upload third-party scripts. This is critical when working on systems where security controls block offensive tools.

**Operational Note:** This process is significantly slower and less efficient than PowerView, especially in large environments.

1. **Generate User List:** Export all domain usernames to a text file to serve as a target list.
2. **Iterative Filtering:** Loop through the list and query the ACL for each user, filtering for the identity you control.

|Command|Purpose|
|:--|:--|
|`Get-ADUser -Filter * \|Select-Object -ExpandProperty SamAccountName > .txt`|
|`foreach($line in [System.IO.File]::ReadLines("<PATH_TO_FILE>")) {get-acl "AD:$(Get-ADUser $line)" \|Select-Object Path -ExpandProperty Access \|

---

### Group Nesting and Transitive Control

Attackers must analyze how permissions propagate through group memberships. Rights are inherited transitively: if a user controls **Group A**, and **Group A** is a member of **Group B**, the user inherits the rights of **Group B**.

**Workflow for Path Analysis:**

1. **Identify Initial Control:** Find an object the user can manipulate (e.g., a group with `GenericWrite`).
2. **Check Nesting:** Determine if that group is a member of other groups using `Get-DomainGroup`.
3. **Trace the Chain:** Follow the nesting until you identify a group that grants high-value rights (e.g., `GenericAll` over a privileged user).

| Command                                                      | Purpose |
| :----------------------------------------------------------- | :------ |
| `Get-DomainGroup -Identity "<GROUP_NAME>" \|select memberof` |         |

---

### Graphical Analysis (BloodHound)

BloodHound is used to visualize complex attack paths that are difficult to track manually.

- **First Degree Object Control:** Rights held directly by the user over an object.
- **Transitive Object Control:** The full attack chain, including rights inherited via group memberships across multiple "hops".
- **Abuse Help:** Right-clicking an **edge** (the relationship line) provides specific instructions and commands for abusing that ACL.

---

### Attack Implications & Unlocks

The following rights allow for escalation or specific lateral movement techniques:

| Right                          | Attack Implication / Unlock                                                                   |
| :----------------------------- | :-------------------------------------------------------------------------------------------- |
| **User-Force-Change-Password** | Allows resetting a target's password without knowing the current one.                         |
| **GenericWrite**               | Allows adding any user (or yourself) to a group to inherit its rights.                        |
| **GenericAll**                 | Grants **full control** over the target object (User, Group, or Computer).                    |
| **DS-Replication-Get-Changes** | Provides the necessary permissions to perform a **DCSync attack** to dump domain credentials. |