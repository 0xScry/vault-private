### **MSSQL: Stealing NTLMv2 Hashes via xp_dirtree**

#### **Overview**

This technique leverages an undocumented MSSQL stored procedure, **xp_dirtree**, to force the MSSQL service account to authenticate against an attacker-controlled share,. This is not a direct exploit of a CVE but a misuse of the **SMB authentication mechanism**.

#### **When to Use**

- When you have **direct interaction** with an MSSQL server or a vulnerable web application.
- When you need to obtain the **NTLMv2 hash** of the service account running MSSQL for offline cracking or lateral movement,.

---

#### **Vulnerability Mechanism: xp_dirtree**

The `xp_dirtree` function is designed to view the contents of a local or remote folder. When the function is directed to a network share, the Windows host automatically attempts to authenticate, sending the service account's **NTLMv2 hash** to the destination.

|Parameter|Description|
|:--|:--|
|**Folder/Path**|The local or remote (UNC) path to be queried.|
|**Depth**|Determines how many subfolder levels the function should traverse.|
|**Target Folder**|The specific directory to be inspected.|

---

#### **Attack Implications**

- **Offline Cracking:** Captured NTLMv2 hashes can be cracked to recover the cleartext password.
- **SMB Relay:** The hash can be "replayed" to log into other network systems where the MSSQL account has **local admin privileges**.
- **Lateral Movement:** While Microsoft has patched SMB relay back to the originating host, gaining admin access on a different host may allow for further credential harvesting to eventually compromise the original system.

---

#### **Operational Workflow**

1. **Preparation:** Start a capture tool on the attack machine to intercept incoming SMB authentication attempts.
2. **Execution:** Call the `xp_dirtree` function via a direct SQL connection or a vulnerable web application, specifying the attack machine as the destination,.
3. **Authentication:** The MSSQL service executes the command with **elevated privileges**, sending the authentication hash to the SMB service on the attacker's host.
4. **Interception:** Use a protocol analyzer or specialized tool to capture and display the hash.

---

#### **Command and Tool Reference**

|Category|Tool / Component|Role in Attack|
|:--|:--|:--|
|**Source**|MSSQL User Input|Specifies the `xp_dirtree` function and the remote attacker share.|
|**Process**|MSSQL Service|Executes the command and queries the specified folder.|
|**Interception**|`Responder`|Intercepts and displays the NTLMv2 hash.|
|**Interception**|`Wireshark` / `TCPDump`|Captures the network traffic containing the hash for analysis.|
|**Destination**|SMB Service|The attacker-controlled host where authentication is directed.|

**Note on Command Execution:** Beyond `xp_dirtree`, other methods for executing commands within MSSQL include running **Python code** directly in a SQL query, though these require specific configurations discussed in other modules.

---

#### **Required Privileges**

- Executing system-level commands like `xp_dirtree` requires the MSSQL service to run with **elevated privileges**.