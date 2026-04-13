### User Enumeration for Password Spraying

#### Initial Requirement: Domain Password Policy

Before initiating a password spray, you must determine the **domain password policy**. This information allows you to calculate the number of attempts allowed before triggering a lockout and defines the required wait time between spray cycles.

**Strategy when policy is unknown:**

- **Request the policy** directly from the client.
- Perform a single **"hail mary" attempt** with one highly targeted password if all other foothold options are exhausted.
- Execute a **slow spray** (e.g., once every few hours) to minimize the risk of locking out accounts.
- **Maintain activity logs:** Always document attempts to avoid redundant effort and to provide a cross-check for the client if security systems flag suspicious activity.

|Policy Component|Operational Impact|
|:--|:--|
|**Minimum Length / Complexity**|Informs the selection of passwords for the spray list.|
|**Account Lockout Threshold**|Defines the maximum number of attempts before accounts are disabled.|
|**Bad Password Timer**|Sets the specific duration to wait between spray attempts to reset failure counters.|

---

#### Internal Enumeration Methodologies

##### 1. Unauthenticated SMB & LDAP Enumeration

**Scenario:** Use when you have internal network access but **no valid domain credentials**. **Goal:** Leverage misconfigured Domain Controllers (DCs) to extract the full AD user list and password policy.

**Operational Workflow:**

1. **Check for Misconfigurations:** Test for **SMB NULL sessions** or **LDAP anonymous binds** on the target Domain Controller.
2. **Retrieve User List:** Use enumeration tools to dump the directory.
3. **Clean Output:** Filter the results to produce a list containing only usernames, one per line.
4. **Audit Lockout Counters:** Use **CrackMapExec** to view `badpwdcount` and `badpwdtime`.
    - _Decision Point:_ In multi-DC environments, query the DC with the **PDC Emulator FSMO role** or aggregate counts from all DCs to ensure accuracy.

|Tool|Technique|Command|
|:--|:--|:--|
|**enum4linux**|SMB enumeration with output filtering.|`enum4linux -U <TARGET_IP> \|
|**rpcclient**|Connect anonymously to run `enumdomusers`.|`rpcclient -U "" -N <TARGET_IP>`|
|**CrackMapExec**|SMB enumeration with lockout status tracking.|`crackmapexec smb <TARGET_IP> --users`|
|**ldapsearch**|Manual LDAP query (requires search filter).|`ldapsearch -h <TARGET_IP> -x -b "DC=,DC=LOCAL" -s sub "(&(objectclass=user))" \|
|**windapsearch**|Automated LDAP enumeration via anonymous bind.|`./windapsearch.py --dc-ip <TARGET_IP> -u "" -U`|

##### 2. Kerberos User Enumeration (Kerbrute)

**Scenario:** Use for **high-speed enumeration** when no other internal access is available. **How it works:** Kerbrute sends TGT requests without Pre-Authentication.

- **Valid User:** KDC prompts for Pre-Authentication.
- **Invalid User:** KDC responds with `PRINCIPAL UNKNOWN`.

**Operational Impact:**

- **Stealth:** Enumeration via this method **does not** trigger Windows Event ID 4625 (Logon Failure) and **cannot** cause account lockouts.
- **Detection:** Generates **Event ID 4768** (TGT Requested), which can be monitored if Kerberos event logging is enabled via Group Policy.
- **Wordlists:** Recommended lists include `jsmith.txt` (flast format) or those from the `statistically-likely-usernames` repository.

| Tool         | Command                                                          |
| :----------- | :--------------------------------------------------------------- |
| **kerbrute** | `kerbrute userenum -d <DOMAIN> --dc <TARGET_IP> <WORDLIST_PATH>` |

##### 3. Credentialed & SYSTEM Enumeration

**Scenario:** Use when you possess **valid domain credentials** or **SYSTEM access** on a domain-joined host. **Attack Unlock:** SYSTEM access allows you to impersonate the computer object, which functions as a domain user account for directory queries.

|Tool|Command|
|:--|:--|
|**CrackMapExec**|`sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --users`|

---

#### External Enumeration (Last Resort)

**Scenario:** Use when all internal enumeration methods are unavailable. **Goal:** Construct a baseline user list from public sources to gain an initial foothold.

**Operational Workflow:**

1. **Email Harvesting:** Identify the organization's email naming convention.
2. **LinkedIn Scraping:** Use `linkedin2username` to generate potential usernames from employee profiles.

---

#### Dangerous Misconfigurations

|Setting|Risk|Attack Implication|
|:--|:--|:--|
|**SMB NULL Session**|Permits unauthenticated access to the `IPC$` share.|Allows anyone to dump the entire domain user list and password policy.|
|**LDAP Anonymous Bind**|Permits directory queries without credentials.|Enables unauthenticated retrieval of all Active Directory objects.|
|**Disabled Kerberos Logging**|Fails to log TGT requests (Event 4768).|Allows high-speed Kerbrute enumeration to bypass SIEM/defender detection.|