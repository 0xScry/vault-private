### **Pivoting, Tunneling, and Port Forwarding: Conclusion & Next Steps**

#### **Operational Significance**

A deep understanding of **Pivoting**, **Tunneling**, **Port Forwarding**, and **Lateral Movement** is essential for everyday penetration testing duties. Results from these actions often dictate the **next steps** for an entire assessment team.

#### **Skill Remediation & Advancement**

Use the following table to determine when to revisit foundational material or advance to specific specializations based on performance in this module:

|Goal / Requirement|Recommended Module/Resource|Why it Matters|
|:--|:--|:--|
|**Foundation Remediation**|Introduction to Networking|Necessary if terminology, subnetting, or Layer 2-3 concepts were challenging.|
|**Enterprise Pentesting**|Intro to AD / AD Enumeration and Attacks|Critical for applying pivoting skills in **Active Directory** environments.|
|**Payload Improvement**|Shells and Payloads|Enhances exploitation skills and payload insight within target networks.|
|**Web-Based Pivoting**|Intro to Web Apps / File Upload Attacks|Clarifies techniques for when webserver shells are used as pivot points.|

---

#### **Practical Application & Lab Selection**

Practice provides exposure to various attack paths and builds comfort with pivoting tools.

|Lab / Resource|Type|Focus / Context|
|:--|:--|:--|
|**Starting Point**|HTB Labs|Foundational practice for applying module skills.|
|**Dante**|Pro Lab|**Chaining pivoting skills** with other enterprise attack knowledge in a simulated corporate network.|
|**Offshore / RastaLabs**|Pro Lab|Intermediate-level environments for **network pivoting** practice.|
|**Ascension**|Mini Pro Lab|Extreme challenge involving **two different AD domains**.|
|**Ippsec / 0xdf**|Walkthroughs|Understanding how tools and tactics tie into a **holistic attack path**.|

---

#### **Advanced Technical Resources**

Refer to these sources for specific techniques and engagement-ready documentation:

|Resource|Key Focus Area|
|:--|:--|
|**SpecterOps**|SSH Tunneling and **proxies over multiple protocols**.|
|**RastaMouse**|C2 infrastructure, payloads, and red-teaming.|
|**Plaintext’s Workshop**|Specific workshop for **Pivoting**.|
|**SANS Webcasts**|Various pivoting tools and diverse **avenues of use**.|

---

#### **Defensive Implications**

For defenders, identifying pivoting is critical to halting lateral movement.

- **Detection Focus:** Monitor for hosts being used as **pivot points**.
- **Traffic Analysis:** Spot compromised hosts by identifying traffic being routed through **non-standard routes**.

_**Note:** The provided source text focuses on methodology, resource mapping, and next steps; it does not contain specific command-line syntax for the techniques mentioned._