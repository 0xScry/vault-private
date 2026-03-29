# WHOIS Reconnaissance

**WHOIS** is a query and response protocol used to access databases storing information about registered internet resources, including **domain names**, **IP address blocks**, and **autonomous systems**.

### Methodology: Why WHOIS Matters

During the **reconnaissance phase**, WHOIS data provides insights into an organization's digital footprint and potential vulnerabilities. It functions as a directory to identify the owners or parties responsible for specific online assets.

---

### Tool Installation

Before performing lookups, ensure the utility is installed on your Linux-based attack machine.

1. **Update local package index**:
    
    ```
    sudo apt update
    ```
    
2. **Install the whois package**:
    
    ```
    sudo apt install whois -y
    ```
    

---

### Operational Workflow: Executing Lookups

The primary method for gathering this data is through the command-line interface.

|Command|Purpose|
|:--|:--|
|`whois <DOMAIN>`|Performs a lookup for the specified domain to retrieve registration and ownership details.|

**Execution Example:**

```
whois <DOMAIN>
```

---

### Analysis and Scenario Context

The value of WHOIS data depends on the investigation context. Analysts use results to identify **red flags** or map out **threat actor infrastructure**.

#### 1. Phishing Investigation

Use WHOIS when an email security gateway flags suspicious links.

- **Goal:** Determine if a domain is part of a malicious campaign.
- **Red Flags:**
    - **Recent registration dates** (indicates temporary infrastructure).
    - **Hidden registrant information** (privacy shields used to mask identity).
    - **Suspicious hosting providers**.

#### 2. Malware Analysis & C2 Identification

Use WHOIS when a system is infected and communicating with a remote server.

- **Goal:** Gain insights into **Command-and-Control (C2)** infrastructure.
- **Outcome:** Identifying the hosting provider allows for the notification of malicious activity or the discovery of "bulletproof" hosting environments.

#### 3. Threat Intelligence

Use WHOIS to track sophisticated threat groups over time.

- **Goal:** Uncover patterns in registration habits to build a profile of **Tactics, Techniques, and Procedures (TTPs)**.
- **Outcome:** Generates **Indicators of Compromise (IOCs)** that help organizations detect and block future attacks.

---

### Data Points and Interpretation

A standard WHOIS record contains several critical fields for analysis:

|Field|Significance|
|:--|:--|
|**Registrar WHOIS Server**|The server where the domain information is officially maintained.|
|**Creation/Updated Dates**|Helps establish the age and longevity of the domain.|
|**Registrant Organization**|Identifies the legal owner of the domain (e.g., Meta Platforms, Inc.).|
|**Name Servers**|Identifies the DNS infrastructure supporting the domain.|
|**Registrar Abuse Contact**|Provides a channel to report malicious use of the domain.|

**Note on Limitations:** WHOIS records may not directly identify individual employees or specific technical vulnerabilities. It must be combined with other reconnaissance techniques for a comprehensive view of a target's digital footprint.