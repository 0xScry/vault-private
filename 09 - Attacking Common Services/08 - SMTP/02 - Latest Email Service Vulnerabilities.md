### OpenSMTPD Remote Code Execution (CVE-2020-7247)

#### **Overview**

This vulnerability affects **OpenSMTPD** versions up to **6.6.2**. It allows for **unauthenticated Remote Code Execution (RCE)** by exploiting a flaw in how the service records the sender's email address. Because SMTP services typically require **root privileges** to listen on standardized ports (like port 25), the injected commands are executed with **elevated privileges**.

#### **Vulnerability Details**

|Parameter|Detail|
|:--|:--|
|**CVE**|CVE-2020-7247|
|**Impact**|Unauthenticated Remote Code Execution (RCE)|
|**Root Cause**|Improper input validation in the sender's email address recording function|
|**Escape Character**|Semicolon (`;`)|
|**Command Limit**|64 characters|

---

#### **Operational Workflow**

1. **Establish Connection**: Initialize a connection to the SMTP service on the `<TARGET_IP>` via port `<PORT>` (typically 25). This can be done manually or through an automated script.
2. **Compose Email**: Begin the process of composing an email, including the recipient and the message body.
3. **Inject Command**: Insert the desired system command into the **sender field**. The command must be connected to a sender address using a **semicolon (;)**.
4. **Trigger Execution**: Send the data. The OpenSMTPD process reads the information; the **semicolon** interrupts the reading process, leading the system to execute the arbitrary shell command.
5. **Establish Persistence/Access**: Direct the command output or a reverse shell back to the `<ATTACK_IP>` to gain remote access to the system.

---

#### **Command Reference**

_Note: While specific protocol syntax (e.g., HELO, MAIL FROM) is not explicitly detailed in the source text, the logic for the injection point is as follows:_

|Input Field|Logic|
|:--|:--|
|**Sender Field**|`<SENDER_ADDRESS>; <SYSTEM_COMMAND>`|

---

#### **Attack Implications**

- **Privilege Escalation**: Because the service runs with root privileges to bind to port 25, any child processes or injected commands inherit these **elevated privileges**.
- **System Access**: Successful exploitation typically unlocks a network connection back to the attacker's host, providing full access to the target Linux distribution.

---

#### **Relevant Practice Labs (Hack The Box)**

Use these targets to practice SMTP-based attack vectors including brute-forcing, phishing, and enumeration:

|Box Name|Attack Techniques Mentioned in Source|
|:--|:--|
|**Rabbit**|Brute-forcing Outlook Web Access (OWA), Phishing with malicious macros|
|**SneakyMailer**|Phishing, Inbox enumeration via IMAP/Netcat|
|**Reel**|SMTP user brute-forcing, Phishing with malicious RTF files|

_Additional research for these techniques can be found via **ippsec.rocks** to find specific walkthroughs or video demonstrations._