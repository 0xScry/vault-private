# Credentialed Windows Enumeration

Credentialed enumeration from a Windows host identifies **misconfigurations and permission issues** that facilitate **lateral and vertical movement**. While some data is primarily for reporting, identifying issues like the ability to run collection tools or specific user attributes helps improve a customer's security posture.

## 1. ActiveDirectory PowerShell Module

This module is used for **stealthier enumeration** because it consists of built-in cmdlets that blend in with standard administrative activity. It is often found on hosts used by administrators.

### Operational Workflow

1. **Discover Available Modules**: Use `Get-Module` to check for the ActiveDirectory module or custom administrator scripts.
2. **Load the Module**: If not already present in the session, import it to enable the AD cmdlets.
3. **Execute Enumeration**: Query the domain for SIDs, trusts, and group memberships.
4. **Identify High-Value Targets**: Note members of sensitive groups for potential domain takeover.

### Command Reference

|Goal|Command|
|:--|:--|
|**Check for Module**|`Get-Module`|
|**Import Module**|`Import-Module ActiveDirectory`|
|**Basic Domain Info**|`Get-ADDomain`|
|**Find Kerberoastable Users**|`Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName`|
|**Check Trust Relationships**|`Get-ADTrust -Filter *`|
|**List All AD Groups**|`Get-ADGroup -Filter * \|
|**Detailed Group Info**|`Get-ADGroup -Identity "<GROUP_NAME>"`|
|**List Group Members**|`Get-ADGroupMember -Identity "<GROUP_NAME>"`|

---

## 2. PowerView and SharpView

**PowerView** provides deep situational awareness and identifies subtle misconfigurations that automated tools might miss. **SharpView** is a .NET port used for **evasion** when PowerShell usage is hardened or monitored.

### Command Reference

|Goal|Command|
|:--|:--|
|**Enumerate User Details**|`Get-DomainUser -Identity -Domain \|
|**Recursive Group Membership**|`Get-DomainGroupMember -Identity "<GROUP_NAME>" -Recurse`|
|**Map Domain Trusts**|`Get-DomainTrustMapping`|
|**Test Local Admin Access**|`Test-AdminAccess -ComputerName <TARGET_IP>`|
|**Identify SPN Accounts**|`Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName`|
|**SharpView (Help Menu)**|`.\SharpView.exe Get-DomainUser -Help`|
|**SharpView User Enum**|`.\SharpView.exe Get-DomainUser -Identity <USERNAME>`|

- **Attack Implication**: Using the `-Recurse` switch with `Get-DomainGroupMember` reveals users inheriting permissions (like **Domain Admin**) through nested groups, identifying critical targets for privilege escalation.

---

## 3. Automated Share Pillaging (Snaffler)

Snaffler automates the discovery of **sensitive data** (credentials, SSH keys, configuration files) across domain shares, which is more efficient than manual searching in large environments.

### Operational Workflow

1. **Establish Context**: Run Snaffler from a domain-joined host or within a domain-user context.
2. **Host & Share Discovery**: The tool identifies all hosts and then enumerates their shares and readable directories.
3. **Content Hunting**: Snaffler iterates through readable files to find high-value strings.
4. **Review Results**: Analyze the color-coded output or log file for actionable data.

### Execution

```
.\Snaffler.exe -d <DOMAIN> -s -v data -o <LOG_FILE>.log
```

- **-s**: Prints results to the console.
- **-d**: Specifies the target domain.
- **-v data**: Verbosity level that displays only successful findings.
- **-o**: Saves results to a log file for supplemental reporting.

---

## 4. Attack Path Visualization (BloodHound)

BloodHound uses graph theory to reveal complex relationships and hidden attack paths (e.g., Derivative Admin rights).

### Operational Workflow

1. **Data Collection**: Run the **SharpHound** collector on a host using a domain user's context.
2. **Exfiltration/Ingestion**: Transfer the generated ZIP file to the analysis machine and upload it to the BloodHound GUI.
3. **Analysis**: Use the **Analysis** tab to run pre-built or custom Cypher queries.

### SharpHound Collection

```
.\SharpHound.exe -c All --zipfilename <FILENAME> --domain <DOMAIN>
```

- **-c All**: Collects all data points including Groups, Sessions, ACLs, Trusts, and Object Properties.

---

## Dangerous Misconfigurations & Findings

These settings should be prioritized for exploitation and documented in reports.

|Finding|Context / Attack Implication|
|:--|:--|
|**ServicePrincipalName (SPN)**|Indicates the account is susceptible to **Kerberoasting**.|
|**Backup Operators Membership**|Allows members to bypass file system permissions, potentially leading to **domain takeover** if the account is compromised.|
|**Unsupported Operating Systems**|Identifies legacy hosts (e.g., Windows 7, Server 2008) susceptible to old RCE vulnerabilities like **MS08-067**.|
|**Domain Users as Local Admin**|Any account you control can gain administrative access, harvest credentials from memory, or find sensitive data on those hosts.|
|**Overly Permissive Shares**|Risks accidental disclosure of medical, HR, or technical data (IT configuration files, passwords) to standard users.|

- **Edge Case**: In BloodHound, old records of non-existent hosts may appear; always validate if a target is **"live"** before making report recommendations.
- **Failure Condition**: If tools cannot be imported or the environment is restricted to **"Living Off the Land"** techniques, native OS tools must be used instead.