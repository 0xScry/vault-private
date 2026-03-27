# Linux Credential Hunting & Local Privilege Escalation

Hunting for credentials is a primary step after gaining initial access to a Linux system. This process identifies "low-hanging fruit" that can lead to elevated privileges within minutes by locating stored passwords or hashes.

### 1. Methodology: System Context & File Enumeration

Before running commands, identify the **system's role** (e.g., an isolated database server vs. a user workstation). This determines where sensitive data is likely to be stored.

#### Configuration File Discovery

**Goal:** Locate service settings that may contain plaintext credentials or reveal how a service operates.

|Extension|Importance|
|:--|:--|
|`.conf`|Standard configuration file.|
|`.config`|Often used for user-specific or complex application settings.|
|`.cnf`|Commonly used by MySQL/MariaDB and SSL services.|

**Workflow:**

1. **Identify** all configuration files while filtering out noise from system libraries.

```
for l in $(echo ".conf .config .cnf"); do
    echo -e "\nFile extension: " $l;
    find / -name *$l 2>/dev/null | grep -v "lib|fonts|share|core";
done
```

2. **Search** discovered files for specific keywords.

```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc|lib"); do
    echo -e "\nFile: " $i;
    grep "user|password|pass" $i 2>/dev/null | grep -v "#";
done
```

---

### 2. Hunting for Databases, Scripts, and Notes

Credentials are often hardcoded in scripts to allow automated execution without manual password entry.

#### Command Reference: File Types

|Target Type|Why it Matters|Command|
|:--|:--|:--|
|**Databases**|May contain user tables or connection strings.|`find / -name "*.sql" -o -name "*.db" -o -name "*db" -o -name ".db* " 2>/dev/null`|
|**Scripts**|Often contain credentials for API calls or database connections.|`for l in $(echo ".py .pyc .pl .go .jar .c .sh"); do find / -name *$l 2>/dev/null; done`|
|**Notes**|Users may store access points or credentials in unstructured text.|`find /home/* -type f -name "*.txt" -o ! -name "*.*"`|

---

### 3. Scheduled Tasks and Persistence (Cronjobs)

**Scenario:** Use this when looking for automated processes running as higher-privileged users (like root) that might rely on poorly secured scripts or hardcoded credentials.

1. **View system-wide cronjobs:**
    
    ```
    cat /etc/crontab
    ```
    
2. **Enumerate directory-based cronjobs:** (Daily, hourly, monthly, weekly, and Debian-specific `cron.d`).
    
    ```
    ls -la /etc/cron.*/
    ```
    

---

### 4. History and Log Analysis

**Goal:** Extract credentials from past user actions or identify patterns in system authentication.

#### History Files

Users occasionally type passwords directly into the CLI or use `su` with a password in the command history.

```
tail -n 5 /home/<USERNAME>/.bash*
```

#### System Logs

**Scenario:** Use when manual inspection is inefficient; this searches for authentication events, sudo usage, and new user creation.

|Log Path|Description|
|:--|:--|
|`/var/log/auth.log`|Debian-based authentication logs.|
|`/var/log/secure`|RedHat/CentOS authentication logs.|
|`/var/log/mysqld.log`|MySQL server logs.|

**Automated Log Grep:**

```
for i in $(ls /var/log/* 2>/dev/null); do
    GREP=$(grep "accepted|session opened|session closed|failure|failed|ssh|password changed|new user|delete user|sudo|COMMAND=|logs" $i 2>/dev/null);
    if [[ $GREP ]]; then
        echo -e "\n#### Log file: " $i;
        grep "accepted|session opened|session closed|failure|failed|ssh|password changed|new user|delete user|sudo|COMMAND=|logs" $i 2>/dev/null;
    fi;
done
```

---

### 5. Memory, Cache, and Browser Credentials

Applications often store credentials in memory or local caches (like Keyrings) for reuse.

#### Automated Tools

|Tool|Requirement|Function|
|:--|:--|:--|
|**mimipenguin**|`root` permissions|Extracts cleartext credentials from memory.|
|**LaZagne**|Python|Extracts hashes and passwords from browsers, Keyrings, and shadow files.|
|**Firefox Decrypt**|Python 3.9|Specifically targets Firefox's encrypted `logins.json`.|

**Operational Steps for Browser Decryption:**

1. **Locate** the Firefox profile directory.
    
    ```
    ls -l /home/<USERNAME>/.mozilla/firefox/ | grep default
    ```
    
2. **Extract** the `logins.json` file (if manual inspection is needed).
    
    ```
    cat /home/<USERNAME>/.mozilla/firefox/<PROFILE_ID>.default-release/logins.json | jq .
    ```
    
3. **Execute** automated decryption:
    
    ```
    # Using LaZagne
    python3 laZagne.py browsers
    
    # Using Firefox Decrypt
    python3 firefox_decrypt.py
    ```
    

**Attack Implication:** Successfully decrypting browser credentials often unlocks external web accounts (SaaS, internal portals) or corporate VPN/email access.