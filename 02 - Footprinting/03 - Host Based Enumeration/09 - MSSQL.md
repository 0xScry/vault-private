# MSSQL Enumeration and Access

**Microsoft SQL (MSSQL)** is a closed-source relational database management system primarily found on **Windows** systems, particularly those using the **.NET framework**. During an engagement, finding an MSSQL instance often provides opportunities for **privilege escalation** and **lateral movement** if the underlying service accounts or integrated Windows Authentication are compromised.

### Default System Databases

Understanding default databases helps in navigating the server structure once access is gained.

|Database|Description|
|:--|:--|
|**master**|Tracks all system information for an SQL server instance.|
|**model**|Template database for every new database created; changes here reflect in future databases.|
|**msdb**|Used by SQL Server Agent to schedule jobs and alerts.|
|**tempdb**|Stores temporary objects.|
|**resource**|Read-only database containing system objects.|

---

### Dangerous Settings & Misconfigurations

These settings should be prioritized during analysis as they represent high-value attack vectors.

|Setting/Feature|Impact|
|:--|:--|
|**SSMS Saved Credentials**|Vulnerable systems may have **SQL Server Management Studio (SSMS)** installed with saved credentials, allowing direct database access.|
|**Windows Authentication**|Login requests are processed by the local SAM or **Active Directory**. Compromising these accounts allows lateral movement across the domain.|
|**Unenforced Encryption**|By default, encryption is often not enforced when connecting to the SQL service, potentially exposing traffic.|

---

### Footprinting the Service

The goal of footprinting is to identify the **hostname**, **version**, **instance name**, and whether **named pipes** are enabled to inform subsequent exploit or connection attempts.

#### 1. Nmap Enumeration

Use Nmap's specialized MSSQL scripts to gather detailed configuration data from the default port **1433**.

```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=<PORT>,mssql.username=sa,mssql.password=,mssql.instance-name=<INSTANCE_NAME> -sV -p <PORT> <TARGET_IP>
```

#### 2. Metasploit Identification

The `mssql_ping` module is used to quickly scan the service and retrieve server information without a full scripted Nmap scan.

```
msf6 > use auxiliary/scanner/mssql/mssql_ping
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts <TARGET_IP>
msf6 auxiliary(scanner/mssql/mssql_ping) > run
```

---

### Connection and Initial Interaction

#### Locate Connection Tools

On many pentesting distributions, **Impacket's mssqlclient.py** is the preferred tool for remote interaction. Locate it using:

```
locate mssqlclient
```

#### Establishing a Connection

Once credentials are known or guessed, connect to the server to interact via **T-SQL (Transact-SQL)**.

**Operational Workflow:**

1. **Authenticate** to the server using recovered credentials. Use the `-windows-auth` flag if the environment utilizes Active Directory/Windows Authentication.
2. **Verify Access** by listing the available databases to establish a "lay of the land".

**Command Reference:**

|Action|Command|
|:--|:--|
|**Connect (Windows Auth)**|`python3 mssqlclient.py <USERNAME>@<TARGET_IP> -windows-auth`|
|**List Databases**|`SQL> select name from sys.databases`|

**Attack Implications:** Successful authentication allows direct interaction with the **SQL Database Engine**, enabling data exfiltration or further system exploitation depending on the user's permissions.