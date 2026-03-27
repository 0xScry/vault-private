### **SMBGhost (CVE-2020-0796)**

#### **Vulnerability Overview**

**SMBGhost** is a critical **integer overflow** vulnerability within the **SMB v3.1.1** protocol's compression mechanism. It allows an **unauthenticated attacker** to achieve **Remote Code Execution (RCE)** and gain **full access** to a remote target system.

|Parameter|Details|
|:--|:--|
|**CVE**|CVE-2020-0796|
|**Affected Protocol**|SMB v3.1.1|
|**Vulnerable OS**|Windows 10 versions 1903 and 1909|
|**Impact**|Unauthenticated RCE / Full System Access|
|**Privilege Level**|System / Administrator|

---

#### **Technical Requirements & Prerequisites**

Use this technique when the following conditions are met:

- The target is running a vulnerable version of **Windows 10**.
- The SMB server allows requests over **TCP/445**.
- **Compression** is supported and enabled (negotiated during the protocol response phase).
- The SMB driver lacks **bounds checks** to handle the size of data sent during session negotiation.

---

#### **Attack Methodology**

The attack leverages an **integer overflow** triggered when the CPU attempts to generate a number exceeding the allocated memory space for a variable. This occurs during the processing of a **malformed compressed message**.

##### **Operational Workflow**

1. **Initiate Session Negotiation:** The client and server set communication terms over **TCP/445**.
2. **Send Malformed Request:** The attacker sends a **manipulated compressed packet** to the SMB server.
3. **Trigger Integer Overflow:**
    - The server processes the packet with **system/administrator privileges**.
    - Because the data exceeds integer variable limits, it overflows into the **buffer**.
4. **Overwrite CPU Instructions:** The excessive data overwrites subsequent **CPU instructions**, interrupting normal execution.
5. **Instruction Replacement:** The attacker replaces the overwritten buffer with **custom instructions**.
6. **Execute Payload:** The CPU is forced to perform the attacker's tasks using the **SMB server's privileges**.
7. **Establish Remote Access:** The remote system is used as the destination to grant the attacker access to the local system.

---

#### **Attack Implications**

- **Privilege Escalation:** Since the SMB driver operates with **System privileges**, successful exploitation automatically grants the highest level of access on the target.
- **Unauthenticated Entry:** No valid credentials are required to trigger the compression vulnerability during the negotiation phase.
- **Full System Compromise:** Attackers gain the ability to execute arbitrary code, allowing for total control over the destination host.

---

#### **Dangerous Configurations**

| Setting                   | Implication                                                         |
| :------------------------ | :------------------------------------------------------------------ |
| **SMB v3.1.1 Enabled**    | Necessary for the vulnerable compression mechanism to be active.    |
| **TCP/445 Open**          | Allows the initial malformed request to reach the SMB driver.       |
| **Missing Bounds Checks** | The root cause in the SMB driver that permits the integer overflow. |