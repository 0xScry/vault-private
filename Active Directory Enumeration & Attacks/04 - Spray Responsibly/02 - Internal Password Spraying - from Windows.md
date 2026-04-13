# Internal Password Spraying (Windows)

### Methodology & Context

Internal password spraying is conducted after gaining an **initial foothold** on a domain-joined Windows host. The goal is to obtain valid credentials for accounts with **elevated privileges** to enable lateral or vertical movement within the domain,.

**When to use:**

- You have a foothold on a managed Windows device or VM where tools can be loaded.
- You are authenticated to a domain host and need to discover accounts with more rights.
- Inbound connections to your pivot host are restricted, requiring tools to run locally.

---

### Technique 1: DomainPasswordSpray.ps1

This PowerShell tool is the preferred method on domain-joined hosts because it automates user discovery and **lockout prevention** by querying **Active Directory (AD)** directly,.

#### Operational Workflow

1. **Import the module** into the current PowerShell session.
2. **Execute the spray** using a single password against the automatically generated user list.
3. **Confirm the prompt** to initiate the attack after the tool calculates the safest number of attempts based on the domain policy,.
4. **Review the output file** to collect successful username and password combinations.

#### Tool Capabilities

- **Automatic Enumeration:** Generates a user list from AD if authenticated to the domain.
- **Lockout Protection:** Queries the domain password policy and **automatically excludes** accounts within one attempt of their lockout threshold,.
- **Policy Awareness:** Detects if the domain is compatible with **Fine-Grained Password Policies**.

#### Command Reference

|Command|Description|
|:--|:--|
|`Import-Module .\DomainPasswordSpray.ps1`|Loads the tool into the current session.|
|`Invoke-DomainPasswordSpray -Password <PASSWORD> -OutFile <FILENAME> -ErrorAction SilentlyContinue`|Executes the spray and writes successes to a file.|

**Execution Example:**

```
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password <PASSWORD> -OutFile <FILENAME> -ErrorAction SilentlyContinue
```

#### Parameters

|Parameter|Description|
|:--|:--|
|`-Password`|The single password to test against the user list.|
|`-OutFile`|Specifies the file name where successful credentials will be recorded.|
|`-UserList`|(Optional) Used to provide a manual list if the host is not domain-joined or you are not authenticated.|

---

### Technique 2: Kerbrute

**Kerbrute** is an alternative tool used to perform user enumeration and password spraying from a Windows host.

**Execution Example:**

```
C:\Tools\kerbrute.exe passwordspray -d <DOMAIN> --dc <TARGET_IP> <USER_LIST_FILE> <PASSWORD>
```

---

### Risks & Failure Conditions

#### Dangerous Settings

|Setting/Condition|Security & Operational Implication|
|:--|:--|
|**Restrictive Lockout Policy**|If manual administrative intervention is required to unlock accounts, a careless spray can cause a **Denial of Service (DoS)**.|
|**MFA Disclosure**|Some MFA implementations confirm if a credential is valid even if the second factor is not provided, allowing credential reuse elsewhere.|

---

### Mitigations & Detection

#### Defensive Measures

|Technique|Description|
|:--|:--|
|**Multi-factor Authentication (MFA)**|Prevents access even with valid credentials; should be implemented on all external portals.|
|**Principle of Least Privilege**|Restricts application access only to users who require it for their roles.|
|**Network Segmentation**|Isolates compromised subnets to slow or stop lateral movement.|
|**Password Hygiene**|Use of **passphrases** and **password filters** to block common dictionary words or company-specific terms.|

#### Detection (Windows Security Event IDs)

|Event ID|Description|Context|
|:--|:--|:--|
|**4625**|Account failed to log on.|High frequency indicates a standard **SMB/Application** spray.|
|**4771**|Kerberos pre-authentication failed.|Indicates a more sophisticated **LDAP-based** spray.|