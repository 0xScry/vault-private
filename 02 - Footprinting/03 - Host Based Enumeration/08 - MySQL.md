1. Identify open port 3306 and use Nmap NSE scripts to footprint the version and existing users.
2. Manually verify any reported empty passwords or discovered credentials to eliminate **false-positives** from automated scripts.
3. Access the database using the MySQL client if credentials or a login bypass are confirmed.
4. Enumerate the database structure, starting with schema discovery, then table and column identification.
5. If local file system access is achieved, audit configuration files for plain-text credentials and misconfigured security variables.

---

## Footprinting and Service Discovery

**When to use** TCP port 3306 is identified as open during initial network scanning.

Enumerate version, capabilities, and potential users via NSE scripts

```
sudo nmap <TARGET_IP> -sV -sC -p3306 --script mysql*
```

**Gotchas** **False-positives** occur frequently with Nmap reporting a **root account with empty password** when a fixed password is actually required.

## Service Interaction and Authentication

**When to use** Credentials have been harvested from configuration files, brute forcing, or research.

Authenticate to the remote server using the native MySQL client

```
mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>
```

**Dangerous / misconfigured settings**

- Remote access enabled on the public Internet or external networks instead of localhost only.
- Empty passwords for the **root** user or other administrative accounts.

**Gotchas** **Access denied** occurs if there is a space between the `-p` flag and the password.

## Database Enumeration

**When to use** An active session is established and you need to locate sensitive PII, credentials, or administrative data.

List all available databases on the server

```
show databases;
```

Select a target database to set the context for further queries

```
use <DATABASE_NAME>;
```

List tables within the currently selected database to identify data locations

```
show tables;
```

Dump all rows from a specific table to extract data

```
select * from <TABLE_NAME>;
```

Filter table data for specific strings like usernames or administrative roles

```
select * from <TABLE_NAME> where <COLUMN_NAME> = "<STRING>";
```

Display column names and types to understand the table structure

```
show columns from <TABLE_NAME>;
```

**Edge cases**

- The **information_schema** and **sys** databases contain metadata and system catalogs that can reveal the broader environment structure.
- Web applications may return detailed **error descriptions** which reveal internal database interaction logic.

## Configuration Auditing

**When to use** Local shell access or arbitrary file read is available on the database host.

Read the primary configuration file to find plain-text credentials and service users

```
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

**Dangerous / misconfigured settings**

- **user**: Service running as a high-privileged user.
- **password**: Credentials stored in plain-text within the configuration file.
- **admin_address**: Administrative interface listening on reachable IP addresses.
- **debug** or **sql_warnings**: Verbose error output enabled, facilitating SQL injection.
- **secure_file_priv**: Misconfigured or disabled, allowing unauthorized data import/export.

**Gotchas** **Unauthorized access** to the entire database is possible if configuration file permissions allow non-admin users to read plain-text passwords.

> ⚠️ Gap: Nmap scripts may return `ERROR: Script execution failed` if the server uses modern authentication plugins like `caching_sha2_password` that the script cannot handle.