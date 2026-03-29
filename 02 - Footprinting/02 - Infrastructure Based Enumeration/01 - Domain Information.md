### Passive Information Gathering Methodology

The goal of passive information gathering is to understand a company's **Internet presence**, **infrastructure**, and **technologies** without making direct connections that could expose the penetration tester. By navigating as a "customer" or "visitor," you remain hidden while identifying the company's functionality and structure,.

#### 1. Website Scrutiny

Before using technical tools, analyze the company's main website to understand their services.

- **Goal:** Identify what technologies are required to support their specific industry (e.g., IoT, hosting, or data science).
- **Perspective:** Use a **"developer's view"** to infer technical functionality and internal structures from the services they offer.

#### 2. Subdomain Discovery (Certificate Transparency)

SSL certificates often contain multiple subdomains and indicate active services. **Certificate Transparency (CT)** logs provide an audit-proof database of issued certificates.

|Command|Purpose|
|:--|:--|
|`curl -s https://crt.sh/?q=&output=json \|jq .`|
|`curl -s https://crt.sh/?q=&output=json \|jq . \|

#### 3. Infrastructure Identification

It is critical to distinguish between **company-hosted servers** and **third-party providers**. Testing third-party hosts without explicit permission is prohibited.

**Operational Workflow:**

1. Generate a list of subdomains.
2. Resolve subdomains to IP addresses.
3. Filter for IPs belonging to the target organization's infrastructure,.

|Technique|Command|
|:--|:--|
|**Filter Hosted Servers**|`for i in $(cat <SUBDOMAIN_LIST>);do host $i \|
|**Generate IP List**|`for i in $(cat <SUBDOMAIN_LIST>);do host $i \|

#### 4. Service Identification via Shodan

Shodan searches for open TCP/IP ports on devices permanently connected to the Internet. Use this to identify IoT devices, industrial controllers, and server versions without performing an active scan.

|Command|Purpose|
|:--|:--|
|`shodan host <TARGET_IP>`|Query Shodan for open ports, service versions (e.g., nginx, OpenSSH), and location data for a specific IP,.|
|**Bulk Shodan Query**|`for i in $(cat ip-addresses.txt);do shodan host $i;done`|

#### 5. DNS Record Analysis

Analyzing DNS records (A, MX, NS, TXT, SOA) reveals technical insights into the target's environment and third-party integrations,.

- **Command:** `dig any <DOMAIN>`

**Attack Implications of Identified Services:**

|Service Detected|Indicator (TXT/MX Records)|Potential Attack Vector/Unlocks|
|:--|:--|:--|
|**Atlassian**|`atlassian-domain-verification`|Indicates software development/collaboration; check for vulnerabilities or leaked info.|
|**Google Gmail**|`_spf.google.com`|Possible access to open GDrive folders or files via shared links.|
|**LogMeIn**|`logmein-verification-code`|Centralized remote access management; admin access unlocks all managed systems.|
|**Mailgun**|`include:mailgun.org`|Identify API interfaces to test for IDOR, SSRF, or unauthorized POST/PUT requests.|
|**Outlook/O365**|`include:spf.protection.outlook.com`|Potential for Azure blob/file storage access or SMB protocol vulnerabilities.|
|**INWX**|`MS=<ID/USERNAME>`|Hosting provider; "MS" values may reveal the username/ID for the management platform.|

**Infrastructure Decision Point:** Identification of specific IP addresses (e.g., from TXT records or Shodan) marks the transition point where you determine which hosts to prioritize for later **active investigations**.