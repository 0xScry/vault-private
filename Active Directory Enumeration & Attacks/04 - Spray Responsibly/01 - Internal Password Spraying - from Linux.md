# Internal Password Spraying from Linux

Internal password spraying is one of two primary avenues for obtaining **domain credentials** for network access. It involves testing a single password against a list of discovered usernames to identify accounts with weak credentials while avoiding account lockouts.

### **Command Reference**

|Tool|Purpose|Command|
|:--|:--|:--|
|**rpcclient**|Manual spray via Bash loop|`for u in $(cat <USER_LIST>);do rpcclient -U "$u%" -c "getusername;quit" <TARGET_IP> \|
|**Kerbrute**|High-speed Kerberos spray|`kerbrute passwordspray -d <DOMAIN> --dc <TARGET_IP> <USER_LIST> <PASSWORD>`|
|**CrackMapExec**|SMB password spraying|`sudo crackmapexec smb <TARGET_IP> -u <USER_LIST> -p \|
|**CrackMapExec**|SMB hash/local admin spray|`sudo crackmapexec smb --local-auth <NETWORK_CIDR> -u -H <NT_HASH> \|
|**CrackMapExec**|Credential validation|`sudo crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD>`|

---

### **Technique 1: rpcclient (Bash Loop)**

**When to use:** Use when you need to perform an attack from a Linux host without specialized binaries. **Success Indicator:** A valid login is not explicitly messaged; look for the response **Authority Name**.

**Operational Workflow:**

1. Create a wordlist of valid users.
2. Execute a Bash one-liner to iterate through the list.
3. **Filter** the response for **Authority** to isolate successful logins.

---

### **Technique 2: Kerbrute**

**When to use:** Use for rapid validation against the **Domain Controller (DC)**. **Attack Implication:** Interacts with the KDC on **port 88** to identify valid logins quickly.

---

### **Technique 3: CrackMapExec (CME) SMB Spraying**

**When to use:** Use for versatile spraying against single targets or entire subnets. **Operational Workflow:**

1. Run the spray using a username file against a single password.
2. **Filter** the output using `grep +` to find valid hits amidst high-volume output.
3. **Validate** discovered credentials against a Domain Controller to confirm account status and domain details.

---

### **Technique 4: Local Administrator & Hash Spraying**

**When to use:** Use when you have obtained an **NTLM hash** or cleartext password for a local administrator or privileged account. **Context:** Local admin reuse is common due to the use of **gold images** in automated deployments. **Targeting:** Focus on high-value targets like **SQL or Microsoft Exchange** servers, as they often contain highly privileged persistent sessions in memory.

#### **Critical Safety Setting**

|Flag|Impact|Why it Matters|
|:--|:--|:--|
|`--local-auth`|Forces the tool to attempt only one login per machine.|**Prevents account lockout**. Without this, the tool defaults to domain authentication, which can lock out the built-in domain administrator across the network.|

**Attack Implication:** A successful hit (indicated by the **(Pwn3d!)** tag) grants administrative access, allowing for further system enumeration.

---

### **Password Reuse Patterns & Decision Logic**

When a password or hash is discovered, apply these mapping patterns to expand access:

- **Template Mapping:** if a desktop admin uses `<PREFIX>%@admin123`, attempt the same prefix with `<PREFIX>%@server123` against server infrastructure.
- **Administrative Suffixes:** If you obtain the password for `<USERNAME>`, attempt the same password for `<USERNAME>_adm`.
- **Cross-Domain Mapping:** Credentials for a user in `<DOMAIN_A>` may be valid for the same username in `<DOMAIN_B>`.

---

### **Remediation & Detection**

- **Stealth Warning:** Large-scale spraying is **noisy** and not recommended for assessments requiring high stealth.
- **LAPS (Local Administrator Password Solution):** Implementing Microsoft LAPS enforces unique, rotating passwords for every host, neutralizing local admin reuse.