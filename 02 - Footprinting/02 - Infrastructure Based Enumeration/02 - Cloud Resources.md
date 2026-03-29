# Cloud Resource Discovery and Enumeration

Cloud platforms such as **AWS, GCP, and Azure** serve as central management points for modern infrastructure. While providers secure the base infrastructure, **administrator misconfigurations** often leave resources like S3 buckets (AWS), blobs (Azure), and cloud storage (GCP) accessible without authentication.

### Technique: DNS Infrastructure Mapping

Cloud storage endpoints are frequently added to a company's DNS records to simplify employee management and access. Identifying these records reveals the specific cloud providers and instances in use.

**Operational Workflow:**

1. **Resolve** subdomains from a pre-defined list.
2. **Filter** results for the target domain and active IP addresses.
3. **Identify** cloud-specific hostnames (e.g., `amazonaws.com`) within the resolved list.

|Command|Purpose|
|:--|:--|
|`for i in $(cat );do host $i \|grep "has address" \|

### Technique: Google Dorking for Cloud Assets

**Google Dorks** allow for targeted searches of indexed cloud storage files that may contain sensitive company data.

- **When to use:** To find publicly indexed files (PDFs, text docs, code) hosted on cloud providers.
- **Why it matters:** This identifies exposed documents and configurations without direct interaction with the target's infrastructure.

|Platform|Google Dork Syntax|
|:--|:--|
|**AWS**|`intext: "<COMPANY_NAME>" inurl:amazonaws.com`|
|**Azure**|`intext: "<COMPANY_NAME>" inurl:blob.core.windows.net`|

### Technique: Passive Discovery via Third-Party Tools and Source Code

Companies often offload web assets (images, JS, CSS) to cloud storage to reduce web server load, leaving links in the application's **source code**.

1. **Source Code Analysis:** Inspect HTML for `dns-prefetch`, `preconnect`, or direct links to cloud URLs (e.g., `blob.core.windows.net`).
2. **Domain Analysis:** Use tools like `domain.glass` to identify infrastructure details, SSL certificate data, and security measures like **Cloudflare**.
3. **Bucket Specialized Search:** Use **GrayHatWarfare** to search for exposed AWS, Azure, and GCP storage.
    - **Filter** by file format to find high-value targets.
    - **Search** using company name abbreviations often used in naming conventions.

### Attack Implications and Exposed Data

Misconfigured cloud instances can lead to the exposure of highly sensitive credentials that facilitate further exploitation.

|Misconfiguration / Leak|Attack Implication|
|:--|:--|
|**Unauthenticated S3/Blob Access**|Allows unauthorized viewing or downloading of company documents, presentations, and source code.|
|**Exposed SSH Private Keys**|Discovery of files like `id_rsa` allows an attacker to log into machines without a password.|
|**Naming Convention Disclosure**|Using company abbreviations in bucket names makes them easier for attackers to guess and discover via tools like GrayHatWarfare.|

The discovery of a security measure, such as **Cloudflare's** "Safe" assessment, indicates the presence of a gateway layer that must be considered in the overall attack strategy.