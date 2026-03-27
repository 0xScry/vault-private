# Finding Sensitive Information

## Methodology: Information Gathering & Pivot Workflow

The primary goal of service attacks is often to obtain **Remote Code Execution (RCE)** by meticulously collecting and observing every detail, as seemingly insignificant information can facilitate a chain of access.

1. **Initial Enumeration & Anonymous Access**: Attempt anonymous access across all identified services (e.g., FTP, Email, Databases, Storage) to find initial data points or filenames.
2. **Credential Harvesting from Files**: Inspect accessible directories for filenames that may represent potential usernames or passwords.
3. **Cross-Service Credential Testing**: Use discovered strings (e.g., filenames) as credentials against other services, such as **email or web portals**, to establish a foothold.
4. **Internal Information Discovery**: Search compromised communications (emails) for sensitive keywords like "**password**" to identify credentials for high-value targets like **databases**.
5. **Service Exploitation for RCE**: Access the target database and utilize **built-in functionality** to execute system commands, achieving RCE on the server.

## Target Services Overview

These common services are primary targets for discovering sensitive information and automating discovery processes.

|Service|Potential Information Found|Attack Implication|
|:--|:--|:--|
|**FTP**|Filenames, configuration files, anonymous access|Initial username discovery or data leak|
|**Email**|Credentials, internal procedures, business logic|Discovery of high-privilege service passwords|
|**Databases**|Sensitive user data, system configuration|Execution of system commands (RCE)|
|**Storage**|Backups, sensitive documents|Data exfiltration and credential harvesting|

## Target Identification & Analysis

Effective discovery requires understanding the unique nature of the target.

- **Familiarization**: Analyze the target's **business model**, **processes**, **procedures**, and **purpose**.
- **Data Valuation**: Determine what information is essential to the client and what information is helpful to the attack path.

## Dangerous Misconfigurations

These settings allow for the escalation of privileges or the discovery of sensitive data.

| Misconfiguration                 | Impact                                                                                     |
| :------------------------------- | :----------------------------------------------------------------------------------------- |
| **Anonymous Access**             | Allows unauthorized users to view filenames or download sensitive content.                 |
| **Stored Cleartext Credentials** | Email content containing passwords allows attackers to pivot to other services like MSSQL. |
| **Database Command Execution**   | Enabled built-in functionality that allows SQL users to execute OS-level commands.         |