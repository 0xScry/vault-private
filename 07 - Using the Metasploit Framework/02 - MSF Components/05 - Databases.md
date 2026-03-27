# Metasploit Database Management

Databases in **msfconsole** track results during complex assessments to manage search results, entry points, and discovered credentials. Metasploit uses **PostgreSQL** to provide quick access to scan results and allows the direct configuration of exploit module parameters based on existing findings.

## Initial Configuration

Before using the database, the **PostgreSQL** service must be active and the Metasploit database initialized.

### Operational Workflow: Database Setup

1. **Verify PostgreSQL Status**: Check if the service is running on the host machine.
2. **Start PostgreSQL**: If the service is inactive, start it using **systemctl**.
3. **Initialize the MSF Database**: Create the database user, schema, and configuration file.
4. **Launch and Connect**: Start Metasploit and connect to the database simultaneously.
5. **Verify Connection**: Use internal commands to confirm the database is successfully linked.

### Command Reference

|Command|Purpose|
|:--|:--|
|`sudo service postgresql status`|Checks the current status of the PostgreSQL service.|
|`sudo systemctl start postgresql`|Starts the PostgreSQL RDBMS.|
|`sudo msfdb init`|Initializes the database, users, and schema.|
|`sudo msfdb status`|Checks the initialization status of the MSF database.|
|`sudo msfdb run`|Launches **msfconsole** and connects to the database.|
|`db_status`|Confirms the connection type and status within the console.|

**Note on Troubleshooting**: If initialization fails, ensure Metasploit is updated via `apt update`. If you cannot change passwords or the database is misconfigured, use `msfdb reinit` and restart the PostgreSQL service.

## Managing Workspaces

**Workspaces** function like project folders to **segregate results** by IP, subnet, or domain. This prevents data overlap when conducting multiple assessments or analyzing different network segments.

### Workspace Commands

|Command|Action|
|:--|:--|
|`workspace`|Lists all current workspaces; the ***** indicates the active one.|
|`workspace -a <NAME>`|Adds a new workspace to the database.|
|`workspace <NAME>`|Switches the active context to the specified workspace.|
|`workspace -d <NAME>`|Deletes a specific workspace.|
|`workspace -r <OLD_NAME> <NEW_NAME>`|Renames an existing workspace.|

## Data Ingestion

Information can be added to the database via external file imports or by running tools directly through the console.

### Technique: Importing Scan Results

Use when you have existing scan data from third-party tools like **Nmap**. Metasploit prefers the **.xml** format for imports.

```
msf6 > db_import <FILENAME>.xml
```

- **Implication**: Successfully importing results automatically populates the `hosts` and `services` tables.

### Technique: Direct Scanning

Use `db_nmap` to scan targets without exiting or backgrounding the console.

```
msf6 > db_nmap -sV -sS <TARGET_IP>
```

- **Implication**: Results are **automatically saved** to the current workspace, allowing immediate interaction with discovered services.

## Data Management and Analysis

Once data is ingested, use specialized commands to filter and organize findings.

### Host and Service Management

- **`hosts`**: Displays a table of addresses, hostnames, and OS information. Use `-R` to automatically set the **RHOSTS** variable based on filtered search results.
- **`services`**: Lists discovered ports and protocols. Use `-p <PORT>` to search for specific open services across the database.

### Credential and Loot Tracking

- **`creds`**: Visualizes gathered credentials. You can manually add credentials or filter by service and port.
- **`loot`**: Tracks "owned" data such as hash dumps, passwd files, and shadow files.

|Command Category|Key Parameters|Purpose|
|:--|:--|:--|
|**Credentials**|`add user:<USERNAME> password:<PASSWORD>`|Manually store a discovered login.|
|**Credentials**|`-d -s <SERVICE_NAME>`|Deletes all credentials associated with a specific service (e.g., SMB).|
|**Loot**|`-t <TYPE>`|Search for specific types of loot (e.g., hashes).|

## Data Retention and Backup

Always **backup database content** after a session to prevent data loss from PostgreSQL service failures.

### Exporting Data

```
msf6 > db_export -f xml <FILENAME>.xml
```

- **Why it matters**: Exporting ensures that host lists, loot, and vulnerabilities are preserved for later import or reporting.