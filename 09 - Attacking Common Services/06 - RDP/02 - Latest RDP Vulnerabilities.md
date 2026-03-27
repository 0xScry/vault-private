# **BlueKeep (CVE-2019-0708) - RDP Remote Code Execution**

### **Vulnerability Overview**

**BlueKeep** is a critical vulnerability affecting the **Remote Desktop Protocol (RDP)** service (TCP/3389). It allows for **Remote Code Execution (RCE)** without requiring prior authentication or user interaction.

|Parameter|Value|
|:--|:--|
|**Service/Port**|RDP (TCP/3389)|
|**CVE ID**|CVE-2019-0708|
|**Privilege Level**|**LocalSystem**|
|**Attack Vector**|Pre-authentication via manipulated connection initialization|

---

### **When and Why to Use**

- **When:** Use when an RDP service is identified and the system is likely unpatched (common in large organizations with legacy infrastructure like hospitals).
- **Why:** This technique provides the highest possible privileges (**LocalSystem**) and requires **no credentials** to execute.

---

### **Operational Workflow**

The attack follows a two-cycle process targeting the kernel to achieve execution.

#### **Phase 1: Vulnerability Trigger (Initialization)**

1. **Source:** The attacker sends a **manipulated initialization request** during the settings exchange between client and server.
2. **Process:** The request triggers a function to create a **virtual channel** containing the flaw.
3. **Privileges:** Because RDP is a system administration service, it runs automatically with **LocalSystem Account** privileges.
4. **Destination:** The manipulated function redirects the execution flow to a **kernel process**.

#### **Phase 2: Remote Code Execution**

5. **Source:** A payload is inserted into the process to **free kernel memory** and place attacker instructions.
6. **Process:** The kernel process triggers the **Use-After-Free (UAF)** condition, causing the CPU to point to the attacker's code.
7. **Privileges:** The instructions execute with **LocalSystem Account** privileges since they are running within the kernel.
8. **Destination:** The execution results in a **reverse shell** sent over the network to `<ATTACK_IP>`.

---

### **Attack Implications & Risks**

- **Outcome:** Successful exploitation unlocks a reverse shell with full system control.
- **System Instability:** This exploit is known to cause **system instability** and may result in a **Blue Screen of Death (BSoD)**.

---

### **Decision Points & Critical Warnings**

|Condition|Action / Implication|
|:--|:--|
|**Exploit Stability**|**High Risk.** Can crash the target system.|
|**Engagement Protocol**|**Always consult the client** before running this exploit due to the BSoD risk.|
|**Patch Status**|Microsoft has released updates for current and many older/unsupported versions; ensure the target is unpatched before attempting.|

---

### **Technical Methodology: Use-After-Free (UAF)**

The attack utilizes a **Use-After-Free** technique. After a function is exploited and memory is freed, data is written into the **kernel memory**. This allows the attacker to overwrite the freed memory with specific instructions that the CPU then executes.