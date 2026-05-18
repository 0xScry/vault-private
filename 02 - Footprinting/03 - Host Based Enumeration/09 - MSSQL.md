1. Confirm port 1433/tcp is open to identify a potential MSSQL instance.
2. Execute Nmap scripts to extract the hostname, instance name, versioning, and identify if **named pipes** are active.
3. Use Metasploit for rapid instance pinging if multiple targets are present.
4. Check for **empty passwords** or look for **saved credentials** in SSMS installations on previously compromised hosts.
5. Authenticate via Impacket's mssqlclient using `-windows-auth` if targeting a domain-joined environment or local Windows accounts.
6. Query `sys.databases` to identify user-created vs system databases for further targeting.

---

## Footprinting the Service

Port 1433 is open and you need instance-specific details like version, instance name, and NTLM info.

Comprehensive script scan for versioning and common misconfigurations

```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <TARGET_IP>
```

Identify basic server and instance names quickly via Metasploit

```
use auxiliary/scanner/mssql/mssql_ping
set rhosts <TARGET_IP>
run
```

- Nmap scripts -> `ms-sql-info` -> prefer for detailed footprinting including NTLM and config checks.
- Metasploit -> `mssql_ping` -> prefer for fast identification of instance names and ports without full script overhead.

**Dangerous / misconfigured settings**

- **Named pipes** enabled, allowing alternative communication channels.
- Active **Direct Access Connection (DAC)** ports exposed on 1434.

**Gotchas** **Connection failed** on the DAC port 1434 is common even if the service is otherwise accessible.

## Remote Connection and Authentication

Credentials have been obtained or guessed and remote T-SQL interaction is required.

Locate the client script path on a Linux distribution

```
locate mssqlclient
```

Authenticate to the instance using Windows Authentication

```
python3 mssqlclient.py <USERNAME>@<TARGET_IP> -windows-auth
```

**Dangerous / misconfigured settings**

- **Encryption not enforced** by default during the connection attempt.
- **Windows Authentication** used with domain accounts, which can facilitate lateral movement if the account is compromised.
- **Saved credentials** within the SSMS client application on compromised workstations.

**Edge cases**

- SSMS is a client-side app; look for it on any admin or developer system, not just the database server.

**Gotchas** The SQL service often runs as **NT SERVICE\MSSQLSERVER**, which defines its local permissions.

## Database Enumeration

Session established and you need to map the database structure.

List all databases on the instance to find non-default targets

```
select name from sys.databases
```

**Dangerous / misconfigured settings**

- `master` database tracks all system information and can reveal sensitive environment details.
- `msdb` contains jobs and alerts scheduled by the SQL Server Agent.

**Gotchas** The **resource** database is read-only and contains system objects; do not expect to write to it.