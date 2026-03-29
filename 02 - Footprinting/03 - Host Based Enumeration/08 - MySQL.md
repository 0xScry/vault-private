### MySQL Service Fundamentals & Footprinting

**MySQL** is an open-source relational database management system (RDBMS) based on the **client-server principle**. It is a core component of **LAMP** (Linux, Apache, MySQL, PHP) and **LEMP** (Nginx) stacks, serving as the central repository for sensitive data such as usernames, passwords, and site content.

---

### Dangerous Configurations

Misconfigurations in the MySQL configuration file (typically `/etc/mysql/mysql.conf.d/mysqld.cnf`) can lead to unauthorized access or information disclosure.

|Setting|Security Implication|
|:--|:--|
|`user`|Defines the system user the service runs as; security-relevant if permissions are mismanaged.|
|`password`|Often stored in **plain text** within configuration files.|
|`admin_address`|The IP listening for administrative connections; if exposed, allows remote management attempts.|
|`debug`|Provides verbose information during errors, which can be used to map the database structure.|
|`sql_warnings`|Controls whether INSERT statements produce information strings, potentially leaking data.|
|`secure_file_priv`|Limits data import/export operations; if misconfigured, may allow reading or writing files to the system.|

**Attack Implication:** If an attacker gains file-read access (e.g., via LFI) or a shell, they can read the configuration file to harvest **plain-text credentials**. Verbose error messages from `debug` or `sql_warnings` often confirm **SQL Injection** vulnerabilities and can be manipulated to execute system commands.

---

### Methodology: Footprinting the Service

#### 1. Service Scanning

Use **Nmap** to identify the service version and run default vulnerability scripts. MySQL typically runs on **TCP port 3306**.

```
sudo nmap <TARGET_IP> -sV -sC -p3306 --script mysql*
```

- **Goal:** Determine the exact version and identify common misconfigurations like **empty passwords** or **anonymous access**.
- **Decision Point:** Always manually verify Nmap script results. Scripts may report **false-positives**, such as claiming the `root` account has an empty password when it is actually protected.

#### 2. Remote Interaction

If credentials are found or guessed, attempt a remote connection using the `mysql` client.

```
# Attempt login without a password
mysql -u <USERNAME> -h <TARGET_IP>

# Attempt login with a password (no space after -p)
mysql -u <USERNAME> -p<PASSWORD> -h <TARGET_IP>
```

---

### Database Enumeration

Once connected, follow this workflow to extract sensitive information and understand the environment.

1. **Identify Available Databases:** Locate system and user-defined databases.
2. **Select a Target:** Switch context to a specific database (e.g., `mysql`, `sys`, or `information_schema`).
3. **Map Tables and Columns:** Determine where sensitive data is stored.
4. **Extract Data:** Query the rows for credentials or configuration details.

#### Command Reference

|Command|Goal|
|:--|:--|
|`show databases;`|Lists all databases accessible to the current user.|
|`use <DATABASE>;`|Switches the session to the specified database.|
|`show tables;`|Displays all tables within the selected database.|
|`show columns from <TABLE>;`|Lists field names and data types for a specific table.|
|`select * from <TABLE>;`|Dumps all data from the specified table.|
|`select * from <TABLE> where <COLUMN> = "<STRING>";`|Filters data to find specific entries (e.g., a specific username).|
|`select version();`|Retrieves the specific database engine version.|

**Note on System Metadata:**

- **`information_schema`**: Contains metadata about all other databases.
- **`sys` (System Schema)**: Contains management metadata, including host summaries and user statistics.

```
-- Example: Identifying unique users and their connection hosts
select host, unique_users from host_summary;
```