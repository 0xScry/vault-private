### **Subdomain Takeover**

#### **Overview**

A **subdomain takeover** occurs when a company cancels a third-party service (e.g., AWS, GitHub) but fails to remove the associated **DNS record**. This leaves a **CNAME** record pointing to a non-existent external domain that an attacker can register to gain control over the subdomain.

---

#### **Vulnerability Identification**

|Condition|Indicator|Why it Matters|
|:--|:--|:--|
|**DNS Record**|Existing **CNAME** record pointing to a third-party provider.|The link between the company domain and the service provider still exists.|
|**Service Status**|Subdomain returns an **HTTP 404** error.|Confirms the third-party service is inactive or the external bucket/page no longer exists.|
|**Availability**|Third-party provider allows anyone to register the orphaned name.|Enables an attacker to claim the endpoint and host malicious content.|

---

#### **Operational Workflow**

**Phase 1: Initiation and Setup**

1. **Identify** a subdomain no longer in use by the target company.
2. **Verify** if the **CNAME** record points to a third-party provider that is no longer occupied.
3. **Register** the non-existent domain or resource on the third-party provider’s platform.
4. **Link** your own server/content to the newly registered resource at the provider.

**Phase 2: Execution and Forwarding**

1. A **victim** enters the URL of the affected subdomain into their browser.
2. The browser utilizes the **outdated CNAME record** from the company's DNS server.
3. The **DNS server** redirects the user to the attacker-controlled subdomain because it still considers the entry **trustworthy**.
4. The victim is forwarded to the **attacker's server** (the destination), where they interact with malicious content.

---

#### **Attack Implications**

Gaining control of a legitimate subdomain unlocks several high-impact attack vectors:

- **Phishing:** Launching campaigns from an official domain (e.g., `<SUBDOMAIN>.<DOMAIN>`) to exploit user trust.
- **Cookie Stealing:** Bypassing security boundaries to capture session data.
- **CSRF/CORS Abuse:** Misusing cross-origin resource sharing or forging requests from a trusted origin.
- **Bypassing Security Policies:** Defeating **Content Security Policy (CSP)** by hosting scripts on a whitelisted subdomain.

---

#### **Misconfigured Settings Table**

|Misconfiguration|Impact|
|:--|:--|
|**Orphaned CNAME Records**|Directs traffic to external domains that can be claimed by unauthorized parties.|
|**Inactive 3rd Party Services**|Leaves subdomains pointing to "dead" AWS buckets or GitHub pages.|
|**Trust in DNS Entries**|DNS servers forward visitors to hijacked subdomains because the entry remains in the authorized list.|

---

#### **Technique Selection Context**

- **When to use:** Use this technique when you find subdomains returning **HTTP 404** errors or pointing to inactive service providers.
- **Why it works:** It exploits the fact that companies often cancel paid services but forget to delete the "free" DNS entries associated with them.
- **Automation:** Discovery and **Proof of Concept (PoC)** generation can be automated using specialized tools available on GitHub.