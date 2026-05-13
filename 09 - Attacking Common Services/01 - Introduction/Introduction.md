# Methodology

1. **Service Identification**: Determine if the target is **SMB**, **Email** (SMTP/IMAP), or a **Database** (MSSQL/MySQL)
2. **Access Method**:
    - **SMB (Windows)**: Use `net use` or `New-PSDrive` to map shares for local-like interaction
    - **SMB (Linux)**: Use `mount -t cifs` to attach shares to the local filesystem
    - **Databases**: Use `sqsh` (MSSQL/Linux), `sqlcmd` (MSSQL/Windows), or `mysql` binaries
    - **Email**: Configure `evolution` for IMAP/SMTP access
3. **Data Extraction**:
    - **File Search**: Use `dir /s`, `Get-ChildItem -Recurse`, or `find` to locate sensitive filenames like **credentials** or **secrets**
    - **Content Search**: Use `findstr`, `Select-String`, or `grep` to identify patterns within files
    - **Database Enumeration**: Use Transact-SQL statements via CLI or GUI (dbeaver) to extract tables

---

## SMB Interaction (Windows)

Accessing remote shares when local GUI is unavailable or automated file discovery is required

List remote share contents anonymously or with current session tokens

```
dir \\<TARGET_IP>\<SHARE_NAME>\
```

Map a remote share to a local drive letter for persistent CLI access

```
net use <DRIVE_LETTER>: \\<TARGET_IP>\<SHARE_NAME>
```

Authenticate to a share using specific credentials and map it

```
net use <DRIVE_LETTER>: \\<TARGET_IP>\<SHARE_NAME> /user:<USERNAME> <PASSWORD>
```

Recursive search for specific strings in filenames via CMD

```
dir <DRIVE_LETTER>:*<STRING>* /s /b
```

Count total files in a mapped share and subdirectories

```
dir <DRIVE_LETTER>: /a-d /s /b | find /c ":"
```

Search for a specific pattern inside all files in a mapped share

```
findstr /s /i <PATTERN> <DRIVE_LETTER>:*.*
```

### PowerShell Variants

Map a share using the PowerShell provider

```
New-PSDrive -Name "<DRIVE_LETTER>" -Root "\\<TARGET_IP>\<SHARE_NAME>" -PSProvider "FileSystem"
```

Map a share using a PSCredential object for authenticated access

```
$secpassword = ConvertTo-SecureString <PASSWORD> -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential <USERNAME>, $secpassword
New-PSDrive -Name "<DRIVE_LETTER>" -Root "\\<TARGET_IP>\<SHARE_NAME>" -PSProvider "FileSystem" -Credential $cred
```

Recursive file count in PowerShell

```
(<DRIVE_LETTER>: Get-ChildItem -File -Recurse | Measure-Object).Count
```

Search for text patterns within files using regex matching

```
Get-ChildItem -Recurse -Path <DRIVE_LETTER>:\ | Select-String "<PATTERN>" -List
```

- **Tool Comparison**
    
    - `net use` -> `net use n: \\<IP>\<SHARE>` -> Prefer for legacy CMD environments
    - `New-PSDrive` -> `New-PSDrive -Name "N" -Root "\\<IP>\<SHARE>" -PSProvider "FileSystem"` -> Prefer for complex scripting or passing credential objects
- **Gotchas**
    
    - **Authentication failure** occurs if the share requires credentials and none are provided, triggering a security prompt

---

## SMB Interaction (Linux)

Accessing Windows or Samba shares from a Linux attack host to use native tools like grep or find

Mount a remote SMB share to a local directory using credentials

```
sudo mount -t cifs -o username=<USERNAME>,password=<PASSWORD>,domain=<DOMAIN> //<TARGET_IP>/<SHARE_NAME> <FILE_PATH>
```

Mount a share using a credential file to avoid plaintext passwords in process lists

```
mount -t cifs //<TARGET_IP>/<SHARE_NAME> <FILE_PATH> -o credentials=<FILE_PATH>
```

Structure for the credential file

```
username=<USERNAME>
password=<PASSWORD>
domain=<DOMAIN>
```

Recursive filename search on a mounted share

```
find <FILE_PATH> -name *<STRING>*
```

Recursive case-insensitive string search within files

```
grep -rn <FILE_PATH> -ie <STRING>
```

- **Dangerous / misconfigured settings**
    
    - **Anonymous authentication** allowed on sensitive shares
- **Gotchas**
    
    - **Missing cifs-utils** will cause the mount command to fail; requires manual installation

---

## Database Interaction

Executing SQL queries or system commands after obtaining database credentials

Connect to MSSQL from Linux using sqsh

```
sqsh -S <TARGET_IP> -U <USERNAME> -P <PASSWORD>
```

Connect to MSSQL from Windows using sqlcmd

```
sqlcmd -S <TARGET_IP> -U <USERNAME> -P <PASSWORD>
```

Connect to MySQL from Linux or Windows

```
mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>
```

Install dbeaver for multi-platform GUI database management

```
sudo dpkg -i dbeaver-<VERSION>.deb
dbeaver &
```

- **Tool Comparison**
    
    - `sqsh`/`sqlcmd` -> `sqsh -S <IP> -U <USER>` -> Prefer for quick CLI enumeration or piping output
    - `dbeaver` -> `dbeaver &` -> Prefer for visual exploration of large schemas across multiple database engines (MSSQL, MySQL, PostgreSQL)
- **Edge Cases**
    
    - **High privileges** (e.g., sysadmin) on MSSQL may allow command execution as the service account

---

## Email Interaction

Accessing mailboxes for sensitive information or verifying credentials via mail protocols

Install Evolution mail client on Linux

```
sudo apt-get install evolution
```

- **Gotchas**
    - **Bwrap error** prevents Evolution from starting; bypass by forcing the WebKit sandbox off

```
export WEBKIT_FORCE_SANDBOX=0 && evolution
```

```
- **Encryption mismatch** will prevent connection; use "Check for Supported Types" to verify if TLS or STARTTLS is required
```