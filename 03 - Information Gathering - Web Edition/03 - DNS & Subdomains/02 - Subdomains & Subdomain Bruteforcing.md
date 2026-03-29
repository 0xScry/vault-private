# Subdomain Enumeration

Subdomain enumeration is the systematic process of identifying extensions of a primary domain (e.g., `<DOMAIN>`) to uncover resources and functionalities not directly linked from the main website. These are typically represented by **A** or **AAAA** records mapping names to IP addresses, or **CNAME** records acting as aliases.

### Enumeration Methodologies

The following approaches should be combined for a thorough discovery strategy.

#### 1. Passive Subdomain Enumeration

**Use case:** When stealth is a priority and you want to avoid direct interaction with the target's DNS servers.

- **Certificate Transparency (CT) Logs:** Search public SSL/TLS certificate repositories to find associated subdomains in the **Subject Alternative Name (SAN)** field.
- **Search Engines:** Utilize specialized operators (e.g., `site: <DOMAIN>`) in engines like Google or DuckDuckGo to filter for subdomains.
- **Online Databases:** Query tools that aggregate DNS data from multiple external sources.

#### 2. Active Subdomain Enumeration

**Use case:** When comprehensive discovery is required and detection by the target is not a primary concern.

1. **DNS Zone Transfer:** Attempt to request a complete list of subdomains from the target's DNS server. **Note:** This is rarely successful due to modern security configurations.
2. **Brute-Force Enumeration:** Systematically test a pre-defined wordlist of potential subdomain names against the target domain.

### Active Enumeration Tool Reference

|Tool|Description|
|:--|:--|
|**dnsenum**|Perl-based toolkit for comprehensive DNS reconnaissance, including dictionary and brute-force attacks.|
|**fierce**|Recursive subdomain discovery tool featuring wildcard detection.|
|**dnsrecon**|Versatile tool combining multiple reconnaissance techniques with customizable output.|
|**amass**|Integration-focused tool with extensive data source support.|
|**assetfinder**|Lightweight tool for quick subdomain discovery using various techniques.|
|**puredns**|Flexible DNS brute-forcing tool designed for effective result filtering.|

---

### Operational Workflow: dnsenum

**dnsenum** is used for active reconnaissance to gather DNS infrastructure information and identify valid subdomains via wordlists.

**Step 1: Execute Subdomain Brute-forcing** Test potential subdomain names against the target using a specified wordlist.

```
dnsenum --enum <DOMAIN> -f <WORDLIST_PATH> -r
```

|Parameter|Purpose|
|:--|:--|
|`--enum`|Shortcut for several enumeration operations.|
|`<DOMAIN>`|The target domain to investigate (e.g., `inlanefreight.com`).|
|`-f`|Specifies the path to the wordlist used for brute-forcing.|
|`-r`|Enables recursive lookups.|

**Attack Implications:**

- Successful enumeration identifies the target's **attack surface** by revealing hidden sections like blogs, shops, or mail services.
- Brute-forcing effectiveness relies heavily on the quality of the **wordlist** used.