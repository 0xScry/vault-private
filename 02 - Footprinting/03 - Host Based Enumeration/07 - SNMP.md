### **SNMP Overview**

**Simple Network Management Protocol (SNMP)** is designed to monitor, manage, and remotely configure network devices such as routers, switches, servers, and IoT hardware. It utilizes **UDP port 161** for general communication and **UDP port 162** for **traps**, which are unsolicited data packets sent from the server to the client when specific events occur.

#### **Protocol Versions & Security**

|Version|Security Features|Security Implications|
|:--|:--|:--|
|**SNMPv1**|None.|No built-in authentication or encryption; data is sent in **plain text**.|
|**SNMPv2c**|Community Strings.|Community strings act as passwords but are transmitted in **plain text** and can be easily intercepted.|
|**SNMPv3**|User-based security & encryption.|Uses username/password authentication and **transmission encryption** (via PSK), though it is significantly more complex to configure.|

---

### **Core Concepts: MIB and OID**

- **Management Information Base (MIB):** An independent, standardized tree hierarchy format (ASN.1 based ASCII) used to store device information. It describes what information is available and its data type.
- **Object Identifier (OID):** A unique sequence of integers in dot notation representing a node in the MIB hierarchy. The longer the sequence, the more specific the information.

---

### **Dangerous Configurations**

Misconfigurations in the SNMP daemon (`snmpd.conf`) can grant attackers full access to system information.

|Setting|Attack Implication|
|:--|:--|
|`rwuser noauth`|Provides access to the **full OID tree** without requiring any authentication.|
|`rwcommunity <COMMUNITY_STRING> <IP>`|Provides access to the **full OID tree** regardless of where requests originate.|
|`rwcommunity6 <COMMUNITY_STRING> <IP>`|Same as `rwcommunity`, but applied to **IPv6** addresses.|

---

### **Footprinting Methodology**

#### **1. Identify Community Strings**

Because community strings are often left as defaults (e.g., `public`) or follow predictable patterns based on hostnames, you must identify a valid string to interact with the service. Use this technique when the community string is unknown.

1. Use a wordlist (e.g., from **SecLists**) to brute-force the community string.
2. If the network is large, look for patterns in naming conventions.

```
onesixtyone -c <WORDLIST> <TARGET_IP>
```

#### **2. Enumerate System Information**

Once a community string is obtained, query the OIDs to extract sensitive internal system data, such as OS versions, running processes, or installed software.

1. Query the OID tree using the identified community string.
2. Analyze the output for sensitive strings, such as **installed Python packages** or system contact information.

```
snmpwalk -v2c -c <COMMUNITY_STRING> <TARGET_IP>
```

#### **3. Bulk OID Enumeration**

In scenarios where you need to brute-force or query specific ranges of OIDs rapidly, use tools designed for high-speed querying.

1. Specify the target range (e.g., `.1.3.6.*`) to crawl the tree.

```
braa <COMMUNITY_STRING>@<TARGET_IP>:.1.3.6.*
```

---

### **Command Reference**

|Tool|Purpose|Key Parameters|
|:--|:--|:--|
|**onesixtyone**|Brute-forces community string names.|`-c`: Path to community string wordlist.|
|**snmpwalk**|Queries OIDs for system information.|`-v`: Protocol version; `-c`: Community string.|
|**braa**|High-speed brute-forcing of individual OIDs.|`<STRING>@<IP>:<OID>`: Syntax for targeted querying.|

---

### **Attack Implications**

Successful SNMP footprinting unlocks:

- **System Identification:** Kernel versions, hostnames, and uptime.
- **Software Enumeration:** Lists of installed packages (e.g., `proftpd`, `python3`) which may have known vulnerabilities.
- **Network Intelligence:** Potential exposure of internal IP addresses and configuration details through the MIB.