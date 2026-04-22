### **Kerberos Double Hop Problem Overview**

The **Double Hop** problem occurs when attempting to use **Kerberos authentication** across two or more connections (e.g., Attack Host → Pivot Host → Target Resource).

**Mechanism:**

- **Kerberos tickets** are signed data for specific resources, not passwords.
- In a standard **WinRM/PowerShell** session, the user's **Ticket Granting Service (TGS)** ticket for the target host is sent, but the **Ticket Granting Ticket (TGT)** is not.
- Without a TGT, the pivot host cannot prove the user's identity to a third resource (like a Domain Controller or file share).
- Unlike **PSExec** or **RDP**, which store the **NTLM hash** in memory for reuse, Kerberos-based remote sessions leave credential fields blank.

---

### **Detection and Identification**

Use these methods to confirm if the current session is restricted by the Double Hop issue.

|Method|Command|Observation|
|:--|:--|:--|
|**Check Cached Tickets**|`klist`|Only a ticket for the current server is present; no TGT for the domain is listed.|
|**Inspect Memory**|`.\mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit`|User passwords and NTLM hashes appear as `(null)` or blank.|
|**Test Domain Access**|`get-domainuser -spn` (PowerView)|Returns "An operations error occurred" because the session cannot authenticate to the DC.|

---

### **Workaround #1: PSCredential Object**

**Scenario:** Use when operating from an **evil-winrm** session or a **non-domain joined attack host**. This allows you to manually provide credentials for multi-server requests.

**Operational Workflow:**

1. **Create a Secure String:** Convert the plaintext password into an encrypted string object.
2. **Initialize Credential Object:** Create a `PSCredential` object using the domain username and the secure string.
3. **Execute with Credentials:** Pass the object to cmdlets using the `-credential` parameter to enable the second hop.

**Command Reference:**

|Step|Goal|Command|
|:--|:--|:--|
|1|**Define Secure Password**|`$SecPassword = ConvertTo-SecureString '<PASSWORD>' -AsPlainText -Force`|
|2|**Create Credential Object**|`$Cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USERNAME>', $SecPassword)`|
|3|**Run Multi-Server Command**|`get-domainuser -spn -credential $Cred \|

---

### **Workaround #2: Register PSSession Configuration**

**Scenario:** Use when you have a **Windows attack host** or **RDP access** to a jump host. This is more efficient as it removes the need to pass credentials with every individual command.

**Operational Workflow:**

1. **Register Configuration:** Create a new session endpoint on the pivot host that runs under the context of a specific user.
2. **Restart WinRM Service:** Apply the configuration changes (this will terminate the current session).
3. **Establish New Session:** Connect to the specific named configuration. The local machine now **impersonates** the remote machine in the context of the user, and requests are sent directly to the DC.

**Command Reference:**

|Step|Goal|Command|
|:--|:--|:--|
|1|**Register New Session**|`Register-PSSessionConfiguration -Name <SESSION_NAME> -RunAsCredential <DOMAIN>\<USERNAME>`|
|2|**Apply Changes**|`Restart-Service WinRM`|
|3|**Connect via Configuration**|`Enter-PSSession -ComputerName <TARGET_IP> -Credential <DOMAIN>\<USERNAME> -ConfigurationName <SESSION_NAME>`|

---

### **Attack Implications and Limitations**

- **Unconstrained Delegation:** If enabled on the pivot host, the TGT is automatically sent with the TGS request and cached in memory, bypassing the Double Hop problem entirely.
- **Linux/evil-winrm Limitations:** `Register-PSSessionConfiguration` **fails** in these environments because it requires a GUI credential popup, an elevated PowerShell terminal, and has limitations regarding how Linux handles Kerberos credentials.
- **Network Authentication:** Tools like **evil-winrm** use network authentication, meaning credentials are not stored in memory to authenticate to other resources.

**Dangerous/Misconfigured Settings**

|Setting|Risk / Implication|
|:--|:--|
|**RunAs Session Configuration**|Windows cannot enforce security boundaries between different user sessions using this endpoint.|
|**Unconstrained Delegation**|Allows a server to impersonate any user who authenticates to it, facilitating lateral movement.|