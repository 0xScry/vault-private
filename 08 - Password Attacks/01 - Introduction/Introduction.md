### **Authentication Fundamentals**

#### **Core Security Principles**

Enterprise security relies on a balance between **Confidentiality**, **Integrity**, and **Availability**. This balance is maintained through three primary mechanisms:

- **Auditing and Accounting:** Tracking every file, object, and host.
- **Authentication:** Verifying a user's identity before granting access.
- **Authorization:** Validating that an authenticated user has the appropriate permissions to access specific resources.

#### **Authentication vs. Authorization**

|Concept|Definition|Goal|
|:--|:--|:--|
|**Authentication**|Validation of identity via specific factors.|Establish "who" the user is.|
|**Authorization**|Set of permissions granted to an identity.|Define "what" the user can do.|

---

### **Multi-Factor Authentication (MFA)**

Authentication is the process of validating identity using a combination of factors. The severity of the system determines how many factors are required.

**Common Factors Identified:**

1. **Knowledge:** Something you know (e.g., password, pin-code, email address).
2. **Possession:** Something you have (e.g., Common Access Card (CAC), cell phone for 2FA).
3. **Inherence:** Something you are (e.g., biometrics, fingerprint, facial recognition).

---

### **Password Methodology & Statistics**

The most widespread method of authentication is the **username/password** combination due to the balance of **convenience** and **security**.

#### **Attack Implications & Decision Making**

|Statistic/Trend|Attack Implication|Technique Selection|
|:--|:--|:--|
|**66% Password Reuse**|High probability that a compromised password works on other platforms.|Prioritize **Credential Stuffing** and **Lateral Movement** across different services.|
|**23% High-Volume Reuse**|Significant portion of users reuse passwords across 3 or more accounts.|Target core corporate accounts first to unlock secondary access.|
|**55% Breach Persistence**|Majority of users do not change passwords after a known data breach.|Utilize **Historical Leaks** (e.g., HaveIBeenPwned) for current authentication attempts.|
|**Common Patterns**|24% use weak strings like `123456`, `qwerty`, or `password`.|Use **Password Spraying** with these high-frequency defaults before complex brute forcing.|

#### **User Behavior & Risk Factors**

- **Complexity vs. UX:** Organizations often sacrifice security for user experience; complex MFA or frequent rotation can lead to manual workarounds or user stress.
- **Predictable Content:** 33% of users use children's or pets' names, and 22% use their own names in passwords.
- **Stale Credentials:** Because users fail to rotate credentials after breaches, databases like **HaveIBeenPwned** remain highly effective for identifying vulnerable accounts.

---

### **Operational Workflow: Credential Discovery**

1. **Identify Target Identifiers:** Obtain the user's ID, typically a username or email address.
2. **Check Breach History:** Use services like **HaveIBeenPwned** to determine if the email address appears in historical data breaches.
3. **Analyze Reuse Potential:** If a password is found or guessed, attempt authentication on secondary platforms, accounting for the 66% reuse statistic.
4. **Evaluate Storage:** Proceed to identify how the target environment stores credentials (e.g., encryption or hashing methods) for extraction.