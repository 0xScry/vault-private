## Methodology

1. Perform passive enumeration via **Certificate Transparency** logs and **Search Engine** operators to establish a baseline without alerting SOC.
2. Attempt **DNS Zone Transfers** against target nameservers to identify immediate misconfigurations.
3. Execute **Active Brute-Force Enumeration** using common and custom wordlists to uncover unlinked subdomains.
4. Filter and resolve results to validate high-value targets like dev, staging, or internal-only infrastructure.

---

## Passive Subdomain Enumeration

Targeting external data repositories to identify infrastructure without direct interaction with the target DNS servers.

### Search Engine Discovery

Filtering indexed results to isolate subdomains.

```
site:<DOMAIN>
```

### Certificate Transparency Logs

Extracting subdomains from the **Subject Alternative Name** (SAN) field in public SSL/TLS certificate repositories.

- **Dangerous / misconfigured settings**
    
    - Certificates issued for internal staging or development environments appearing in public logs.
- **Gotchas**
    
    - **Passive enumeration** might not uncover all existing subdomains if they lack public-facing certificates or search indexing.

## DNS Zone Transfer

Directly querying nameservers for a full copy of the zone file.

**When to use** — Initial active phase; requires a misconfigured DNS server allowing unauthorized transfers.

- **Dangerous / misconfigured settings**
    
    - DNS servers permitting global AXFR requests.
- **Gotchas**
    
    - **Security measures** make this technique rarely successful in modern environments.

## Active Subdomain Brute-Force

Systematically testing potential names against the target domain using wordlists.

**When to use** — Target is non-responsive to passive methods or zone transfers; requires a target domain and a wordlist.

### dnsenum

Comprehensive DNS reconnaissance and brute-forcing.

Perform full enumeration including brute-force with a specific wordlist:

```
dnsenum --enum <DOMAIN> -f <FILE_PATH> -r
```

### Tool comparison

- amass
    
    - `amass enum -d <DOMAIN>`
    - Prefer for integration with extensive data sources and other toolsets.
- dnsenum
    
    - `dnsenum --enum <DOMAIN> -f <FILE_PATH>`
    - Prefer for all-in-one Perl-based DNS reconnaissance and dictionary attacks.
- fierce
    
    - Prefer when **wildcard detection** and a simple recursive interface are required.
- dnsrecon
    
    - Prefer for versatile reconnaissance and the need for specific output formats.
- assetfinder
    
    - Prefer for lightweight, fast scans when resource overhead is a concern.
- puredns
    
    - Prefer for high-volume brute-forcing and effective result filtering.
- **Edge cases**
    
    - Use **custom-generated lists** based on identified naming patterns if standard wordlists fail to yield results.
- **Gotchas**
    
    - **Active enumeration** is highly detectable and offers less stealth than passive methods.
    - **Wildcard DNS** can return false positives for every query; use tools with **wildcard detection** like fierce.

> ⚠️ Gap: Source mentions `ffuf` and `gobuster` as active tools but provides no syntax; usage requires external knowledge of DNS-specific flags for these web-fuzzers.