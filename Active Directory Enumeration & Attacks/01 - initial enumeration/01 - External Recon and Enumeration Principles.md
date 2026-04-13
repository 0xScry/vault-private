# External Reconnaissance and Enumeration

External reconnaissance establishes the **technical boundaries** and identifies **information leaks** or **breach data** before starting active testing. This phase ensures a comprehensive test by identifying potential entry points like VPNs, OWA, or cloud portals through passive data gathering.

### Enumeration Principles

Enumeration is an **iterative process** used to find every possible route to the internal network.

1. **Passive Enumeration**: Start with a wide scope using public resources to avoid direct interaction with target infrastructure.
2. **Analyze Results**: Narrow the scope based on discovered data points.
3. **Active Enumeration**: Perform hands-on validation and probing based on passive findings.

---

### External Data Points and Attack Implications

Gathering these items provides the data necessary to build target lists and identify vulnerabilities in the external perimeter.

|Data Point|Description|Attack Implication|
|:--|:--|:--|
|**IP Space**|ASN, netblocks, and cloud hosting providers (AWS/Azure/GCP).|Defines the technical boundary and identifies third-party infrastructure.|
|**Domain Information**|Subdomains and publicly accessible services (Mail, VPN, OWA).|Identifies potential entry points and security defenses like SIEM or IPS.|
|**Schema Format**|Email patterns and Active Directory (AD) username formats.|Used to build valid user lists for **password spraying** and **brute forcing**.|
|**Data Disclosures**|Publicly accessible files and code repositories (GitHub).|Reveals metadata, intranet links, or hardcoded credentials.|
|**Breach Data**|Publicly released usernames and passwords from third-party leaks.|Provides a potential foothold through **credential stuffing** on external portals.|

---

### Infrastructure and Scope Validation

#### 1. ASN and Netblock Identification

Use the **BGP Toolkit** to identify address blocks assigned to an organization.

- **Large Organizations**: Often self-host and possess their own **Autonomous System Number (ASN)**.
- **Smaller Organizations**: Typically host infrastructure in shared spaces (Cloudflare, AWS, etc.).

**Scoping Decision Points:**

- **Shared Infrastructure**: Ensure you are not attacking other organizations sharing the same server or provider.
- **Third-Party Permissions**: Some providers (e.g., Oracle) require a **Cloud Security Testing Notification**, while others (e.g., AWS) have specific testing guidelines. Always confirm **explicit permission** in writing before attacking.

#### 2. DNS Validation

DNS resolution validates the scope and identifies reachable hosts or **subdomains** not disclosed in initial documents. Subdomains on in-scope IP addresses are generally considered valid targets.

|Task|Command|Goal|
|:--|:--|:--|
|**Identify Nameservers**|`nslookup ns1.<DOMAIN>`|Finds the IP of the authoritative nameserver for validation.|
|**Validate IP/Domain**|Use **viewdns.info**|Confirms if an IP matches the target domain and checks IP history.|

---

### Public Data & OSINT Techniques

#### 1. Technical Stack Identification

- **Job Postings**: Sites like LinkedIn or Indeed reveal internal equipment and software versions.
- **Scenario**: A posting for a "SharePoint 2013" admin indicates a likely **legacy vulnerability** if the system was upgraded in place.
- **Public Websites**: "About Us" and "Contact Us" pages provide organizational charts and contact details for schema building.

#### 2. Google Dorking

Use specific search operators to find sensitive files or contact information that define the organizational schema.

|Goal|Command|Why it Matters|
|:--|:--|:--|
|**Find Sensitive Files**|`filetype:pdf inurl:<DOMAIN>`|Documents often contain internal AD username formats in their **metadata**.|
|**Harvest Emails**|`intext:"@<DOMAIN>" inurl:<DOMAIN>`|Identifies naming conventions (e.g., `first.last`) for credential attacks.|

#### 3. Leaks in Code & Cloud Storage

Unintentional data leaks on platforms like **GitHub** or **AWS S3** can provide credentials that bypass the need for brute-forcing.

- **Tools**: Use **Trufflehog** or **Greyhat Warfare** to find hardcoded credentials or notes in code releases.

---

### Username and Credential Harvesting

The goal is to create a high-quality list for gaining a foothold on the internal network.

#### 1. Username Scraping

Use tools to scrape professional networks to generate various username formats based on discovered schemas (e.g., `flast`, `f.last`).

|Tool|Purpose|
|:--|:--|
|**linkedin2username**|Scrapes LinkedIn company pages to build target lists for password spraying.|

#### 2. Breach Data Hunting

Querying breach databases like **Dehashed** or **HaveIBeenPwned** identifies historical credentials.

|Goal|Command|Scenario|
|:--|:--|:--|
|**Query Breach Data**|`sudo python3 dehashed.py -q <DOMAIN> -p`|Use when externally-facing portals (VPN, OWA) utilize AD authentication.|

**Attack Implications**: A single set of low-privilege credentials enables extensive **internal Active Directory enumeration** and advanced attacks.