# Password Spraying Overview

**Password spraying** is a technique used to gain an initial **foothold** or move **laterally** within a network by attempting to log into a service using one common password against a large list of usernames. Unlike brute-forcing, which tries many passwords against a single account, spraying is a **measured attack** designed to minimize the risk of account lockouts.

### When to Use

- **Initial Access:** During the OSINT or initial enumeration phase when services are exposed.
- **Failed Enumeration:** Use when standard checks, such as **SMB NULL sessions** or **LDAP anonymous binds**, fail to provide a list of valid users.
- **Time Management:** Execute sprays while long-running tasks like poisoning or hash cracking are active to maximize efficiency in time-boxed assessments.

---

### Operational Workflow: Targeted Username Generation

If standard username lists (e.g., `jsmith.txt`) or LinkedIn scraping fail to yield results, metadata can reveal internal naming conventions.

1. **Search Public Documents:** Use search engines to find PDFs or office documents published by the organization.
2. **Analyze Metadata:** Inspect the **Author field** in document properties to identify the internal username structure (e.g., GUIDs like `F9L8`).
3. **Generate Permutations:** Use a script to create a list of all possible combinations based on the identified pattern.
4. **Enumerate Valid Users:** Use a tool like **Kerbrute** to verify which generated usernames are valid domain accounts.
5. **Execute Spray:** Attempt a single common password (e.g., `Welcome1`) against the verified user list.

#### GUID Generation Script

This Bash script generates all possible four-character combinations (A-Z, 0-9) for a predictable GUID format.

```
#!/bin/bash
for x in {{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}
do
    echo $x
done
```

---

### Operational Security and Lockout Prevention

Careless spraying can lock out hundreds of production accounts. **Introducing delays** between login attempts is mandatory for OpSec.

|Policy/Setting|Impact on Attack Selection|
|:--|:--|
|**Lockout Threshold**|Often set to **5 failed attempts** before an account is locked.|
|**Auto-Unlock Threshold**|Typically **30 minutes**; some environments require manual administrator resets.|
|**Password Policy**|If known, allows the attacker to stay just under the lockout limit.|

#### Decision Logic for Delay Intervals

- **Known Policy:** If internal access is already established, **enumerate the domain password policy** first to determine the safe frequency of attempts.
- **Unknown Policy:** A good rule of thumb is to **wait a few hours** between spray iterations to allow lockout counters to reset.
- **The "Hail Mary":** If all other options for a foothold are exhausted, perform a **single spray attempt** using one very common password to minimize risk.

---

### Attack Implications

|Success Metric|Impact/Unlocked Techniques|
|:--|:--|
|**High-Accuracy Lists**|Using metadata analysis can identify **100% of domain accounts**, significantly increasing the probability of a successful spray compared to standard lists.|
|**Low-Privileged Access**|Even a low-privileged account enables **BloodHound** to map attack paths toward domain compromise.|
|**Advanced Attack Chains**|Successful sprays can unlock complex chains involving **Resource-Based Constrained Delegation (RBCD)** or **Shadow Credentials**.|