# Active Directory ACL Abuse

**Access Control Lists (ACLs)** are security configurations that define which **security principals** (users, groups, or processes) can access specific resources and their level of access. Misconfigured ACLs are a high-value target because they are **invisible to standard vulnerability scanners** and allow for lateral movement, vertical escalation, and domain compromise.

### Core Components

|Component|Description|
|:--|:--|
|**ACL**|The overall list defining access and auditing for an object.|
|**ACE (Access Control Entry)**|Individual settings within an ACL mapping a principal to specific rights.|
|**DACL (Discretionary ACL)**|Defines actual permissions (e.g., Full Control, Change Password).|
|**SACL (System ACL)**|Defines auditing rules to log access attempts.|

#### ACE Types and Evaluation

ACEs are evaluated **top to bottom** until an **"access denied"** entry is encountered.

|ACE Type|Purpose|Placement|
|:--|:--|:--|
|**Access Denied**|Explicitly blocks access.|DACL|
|**Access Allowed**|Explicitly grants access.|DACL|
|**System Audit**|Records success/failure of access attempts in logs.|SACL|

---

### Dangerous Misconfigurations (BloodHound Edges)

These permissions represent common attack paths that should be prioritized during enumeration.

|Permission / Edge|Attack Implication|
|:--|:--|
|**WriteDacl**|Allows an attacker to modify the DACL to grant themselves additional rights (e.g., GenericWrite).|
|**WriteOwner**|Allows an attacker to take ownership of an object to then modify its DACL.|
|**AddMember**|Allows adding a controlled account to a security group to inherit its permissions.|
|**ReadGMSAPassword**|Enables retrieval of cleartext passwords for Group Managed Service Accounts.|
|**GenericWrite**|Allows updating properties of an object to facilitate account takeover.|
|**Extended Rights**|Includes specialized privileges like **Unexpire-Password** or **Reanimate-Tombstones**.|

---

### Common Attack Scenarios

Techniques are selected based on the specific rights identified during discovery.

|Scenario|Context / Why it Matters|
|:--|:--|
|**Forgot Password Abuse**|Help Desk or IT accounts often have the right to reset passwords. Compromising these allows for **password resets** on high-privileged targets.|
|**Group Membership Management**|Used when a principal has the right to add/remove users. Use this to add a controlled account to a **privileged built-in AD group**.|
|**Excessive User Rights**|Often the result of software installations (e.g., **Exchange**) or legacy configurations that grant unintended rights to standard users.|

---

### Methodology: ACL Enumeration and Abuse

1. **Identify Attack Paths:** Use **BloodHound** to visualize edges between controlled accounts and high-value targets.
2. **Granular Enumeration:** Use **PowerView** or built-in AD management tools to inspect specific ACE entries for the identified objects.
3. **Validate Technique:** Research specific rights if they are uncommon (e.g., extended rights) to determine the exploitation path.
4. **Operational Safety Check:** Determine if the attack is **destructive** (e.g., changing a user's password). **Destructive modifications require written client approval before execution**.
5. **Execution:** Apply the identified abuse technique using appropriate tools (e.g., **GMSAPasswordReader** for GMSA passwords).
6. **Documentation:** Record the attack from start to finish, highlighting every modification made to the AD environment.
7. **Post-Exploitation Cleanup:** **Revert all changes** (e.g., remove added group members) and ensure the client can verify the environment's restoration.

### Tool Reference

|Tool|Primary Use Case in ACL Abuse|
|:--|:--|
|**BloodHound**|Visualization of abusable edges and attack paths.|
|**PowerView**|Enumeration and exploitation of specific ACEs from Windows.|
|**ADUC**|Graphical inspection of DACLs and SACLs via Advanced Security Settings.|
|**GMSAPasswordReader**|Retrieving passwords when the **ReadGMSAPassword** edge is present.|