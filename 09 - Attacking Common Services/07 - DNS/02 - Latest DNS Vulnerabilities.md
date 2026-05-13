## Subdomain Discovery and Identification

**When to use** Target uses third-party providers and has stale DNS records. Active **CNAME records** point to non-existent subdomains or deactivated services (AWS, GitHub) and return an **HTTP 404 error**,.

**Commands** Use automated tools to identify vulnerable subdomains or generate Proof of Concepts.

> ⚠️ Gap: Source lacks specific tool names or CLI syntax for discovery and PoC generation; without external tools, manual verification of millions of records is non-viable.

**Dangerous / misconfigured settings**

- DNS records (CNAME) pointing to deactivated third-party service providers.
- Retaining DNS entries for cancelled services because they incur no additional cost.
- Use of AWS S3 buckets or other cloud resources without deleting the associated subdomain pointers.

**Gotchas** **Subdomain Takeover** will fail if the primary domain administrators remove the outdated DNS record before registration.

---

## Takeover Execution

**When to use** Confirmed **CNAME record** points to a non-existent third-party resource that allows for public registration,.

**Workflow**

1. Identify the unused subdomain name linked via CNAME.
2. Register the non-existent subdomain/resource name directly on the third-party provider's platform.
3. Link the registration to attacker-controlled servers or data sources.

**Commands** Register the identified non-existent domain at the provider to claim the pointer.

```
<THIRD_PARTY_PROVIDER_REGISTRATION_PORTAL>
```

**Edge cases**

- **E-commerce** sector targets are statistically more likely (62%) to be vulnerable.
- Even high-traffic domains (Alexa Top 1000) frequently contain these vulnerabilities.

**Gotchas** **Third-party providers** often do not verify if the person registering the resource is the actual owner of the primary domain.

---

## Post-Exploitation Tactics

**When to use** Successful control of the subdomain is established, allowing the attacker to define the design and function of the page,.

**Impacts**

- **Phishing**: Host malicious login pages on a trusted subdomain (e.g., `<SERVICE_NAME>.<DOMAIN>`) to exploit user trust.
- **Cookie Theft**: Intercept session cookies authorized for the primary domain or its subdomains.
- **Security Policy Bypass**: Defeat Content Security Policy (CSP) or abuse CORS configurations that trust the compromised subdomain.
- **Cross-Site Attacks**: Execute Cross-Site Request Forgery (CSRF) against the primary domain.

**Commands** Mirror the target company's legitimate page to provoke user logins.

```
<ATTACKER_SERVER_CONTENT_HOSTING>
```

**Gotchas** **DNS trust** ensures the visitor's browser considers the attacker-controlled page as an official part of the target domain,.