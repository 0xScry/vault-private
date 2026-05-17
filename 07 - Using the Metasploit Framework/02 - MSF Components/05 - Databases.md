1. Check PostgreSQL service status to ensure the backend is available.
2. Initialize the database schema and user if not already configured.
3. Verify connection status inside the console via `db_status`.
4. Create a dedicated **Workspace** to prevent cross-contamination of engagement data.
5. Ingest target data using external XML imports or direct console scanning.
6. Query `hosts` and `services` tables to filter targets; use `-R` to automatically populate `RHOSTS` for modules.
7. Track gathered credentials and exfiltrated loot.
8. Export the workspace to XML for backup or reporting.

---

## Database Initialization

Need to track results across complex assessments or auto-populate module parameters from findings.

Check if the backend service is running

```
sudo service postgresql status
```

Start the service if inactive

```
sudo systemctl start postgresql
```

Initialize the schema and configuration file

```
sudo msfdb init
```

Launch console with automatic database connection

```
sudo msfdb run
```

- **rake aborted!** typically occurs if Metasploit is outdated; run `sudo apt update` before re-initializing.
- **Database already configured** message indicates the schema exists; check status directly with `sudo msfdb status`.

## Database Troubleshooting

Authentication failures or inability to change the database user password.

Wipe and recreate the database configuration

```
msfdb reinit
```

Manual configuration sync if auto-init fails

```
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
```

Verify connection type is PostgreSQL from within the console

```
db_status
```

- **Connection type: none** means the console is not talking to PostgreSQL; check for the existence of `database.yml` in the expected path.

## Workspace Management

Segregate scan results, hosts, and credentials by project, subnet, or domain.

List all existing workspaces

```
workspace
```

Create and switch to a new project container

```
workspace -a <NAME>
```

Switch active workspace context

```
workspace <NAME>
```

Delete a specific workspace and its associated data

```
workspace -d <NAME>
```

- **Asterisk (*)** indicates the currently active workspace in the list.

## Data Ingestion

Populate the database with target information from scans.

Import Nmap XML or other supported third-party results

```
db_import <FILE_PATH>
```

Execute Nmap and automatically save results to the active workspace

```
db_nmap -sV -sS <TARGET_IP>
```

### Tool comparison

- `db_import`
    
    - `db_import <FILE_PATH>`
    - Prefer when using specialized Nmap flags or importing data from external tools like Nessus.
- `db_nmap`
    
    - `db_nmap <FLAGS> <TARGET_IP>`
    - Prefer for quick scans without leaving the `msfconsole` environment.
- **XML format** is the preferred file type for successful data ingestion.
    

## Target Selection and Filtering

Filtering the database to identify attack surfaces and set module options.

Show up-hosts and set the global RHOSTS variable

```
hosts -u -R
```

Search for specific open ports and set RHOSTS

```
services -p <PORT> --up -R
```

Export current workspace data to a backup file

```
db_export -f xml <FILE_PATH>
```

- **-R flag** is critical for workflow speed; it pushes all filtered results into the `RHOSTS` parameter of the current module.

## Credentials and Loot

Managing collected passwords, hashes, and files exfiltrated from targets.

Add a credential manually to the database

```
creds add user:<USERNAME> password:<PASSWORD> realm:<DOMAIN>
```

Add an NTLM hash to the database

```
creds add user:<USERNAME> ntlm:<HASH>
```

List credentials filtered by a specific service

```
creds -s <SERVICE_NAME>
```

Manually add a local file to the loot repository

```
loot -a <TARGET_IP> -t <SERVICE_NAME> -f <FILE_PATH> -i <INFO>
```

- **creds -d** will **delete** credentials; use with caution when filtering by service.
- **loot -d** will **delete all** loot matching the specified host and type.