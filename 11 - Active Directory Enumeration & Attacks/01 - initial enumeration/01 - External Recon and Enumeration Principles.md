## Methodology

1. **Passive Infrastructure Mapping**: Query BGP Toolkit and DNS registrars to define the target IP space and ASN.
2. **DNS Validation**: Cross-reference historical DNS records with active lookups to confirm live hosts and subdomains.
3. **Public Data Harvesting**: Execute Google Dorks for sensitive filetypes and email patterns; scrape GitHub/S3 for hardcoded secrets.
4. **Credential Scoping**: Scrape LinkedIn for username formatting and query breach databases for cleartext passwords.
5. **Pivot to Active**: Move to internal enumeration once a valid **credential set** is obtained.

---

## Infrastructure Discovery

ASN and netblock identification for public-facing infrastructure. Large orgs often own an **ASN**; smaller targets reside on **shared infrastructure** like Cloudflare or AWS.

Query Hurricane Electric BGP Toolkit for assigned address blocks. `https://bgp.he.net/`

- **BGP Toolkit** → `Search by <DOMAIN> or <TARGET_IP>` → Best for identifying organizational **ASN** and netblocks.

> ⚠️ Gap: Cloud providers like AWS or Oracle often require a **Cloud Security Testing Notification** or adherence to specific guidelines before testing; failure to confirm 3rd-party permissions leads to immediate **engagement breach**.

**Gotchas**: **Shared Infrastructure**. Attacking a shared IP without host-header or tenant-specific targeting can result in hitting out-of-scope organizations.

## DNS Enumeration

Validating scope and identifying undocumented subdomains or mail servers.

Identify subdomains, MX records, and historical IP resolution. `https://viewdns.info/`

Active validation of hostnames found in passive results.

```
nslookup <SERVICE_NAME>.<DOMAIN>
```

- **ViewDNS** → Manual Web GUI → Prefer for historical IP changes and **Reverse IP** lookups.
- **nslookup** → CLI tool → Prefer for real-time validation of nameservers and A records.

**Gotchas**: **Outdated Records**. Passive databases often contain stale info; use **nslookup** to confirm the target currently resolves to an in-scope IP.

## Public Data and File Discovery

Hunting for **information leaks**, intranet links, and **metadata** in published documents.

Search for PDF files hosted on the target domain.

```
filetype:pdf inurl:<DOMAIN>
```

Search for potential email addresses or naming conventions.

```
intext:"@<DOMAIN>" inurl:<DOMAIN>
```

- **Google Dorks** → Web Search → Prefer for finding publicly indexed documents and emails.
- **Trufflehog** → CLI → Prefer for finding hardcoded **credentials** in GitHub repositories.
- **Greyhat Warfare** → Web Search → Prefer for discovering leaked data in **AWS S3 buckets**.

**Gotchas**: **Information Obsolescence**. Job postings or old PDFs may reference **SharePoint 2013/2016** or other legacy versions that have since been patched or decommissioned.

## Username and Credential Harvesting

Building targets for password spraying or finding leaked **cleartext passwords** in breach data.

Generate a username list by scraping LinkedIn employee data. `linkedin2username`

Query breach databases for cleartext passwords or hashes associated with the domain.

```
sudo python3 dehashed.py -q <DOMAIN> -p
```

**Gotchas**: **Stale Credentials**. Breach data passwords often fail against **AD authentication** portals if the leak is old; prioritize creating a **username list** for active spraying.