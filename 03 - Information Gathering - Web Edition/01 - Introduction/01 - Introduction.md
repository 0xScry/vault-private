## Information Gathering Methodology

1. Start with **Passive Reconnaissance** to establish infrastructure footprint and domain ownership while maintaining **minimal detection risk**.
2. Query WHOIS and DNS records to map subdomains and identify administrative contacts.
3. Analyze historical snapshots and public code repositories for **exposed credentials** or deprecated assets.
4. Transition to **Active Reconnaissance** only when direct interaction is required for a comprehensive view of services and vulnerabilities.
5. Execute port scans and service enumeration to identify versions and potential exploit vectors.
6. Perform web spidering to discover hidden resources and map the application structure.
7. Consolidate findings to feed into the **Vulnerability Assessment** phase.

---

## Passive Reconnaissance

**When to use** Establishing initial scope or mapping infrastructure when **detection risk** must be kept at a minimum.

**Commands** Retrieve domain registration and contact details via WHOIS

```
whois <DOMAIN>
```

Analyze DNS records to identify subdomains and mail servers

```
dig <DOMAIN>
```

**Tool comparison**

- DNS Enumeration
    - dig -> `dig <DOMAIN>` -> Standard for detailed record analysis
    - nslookup -> `nslookup <DOMAIN>` -> Alternative for quick resolution
    - host -> `host <DOMAIN>` -> Simple lookup alternative
- External Infrastructure Search
    - Shodan -> Web-based -> Specialized for identifying internet-connected devices
    - Google -> Web-based -> Use for uncovering employee info and social media profiles

**Gotchas** **Less comprehensive** results compared to active methods as it relies solely on publicly accessible data.

> ⚠️ Gap: Standard DNS queries via `dig` or `nslookup` fail to discover **non-public subdomains** or those not listed in public zone files without brute-forcing or zone transfer attempts.

## Active Reconnaissance

**When to use** Direct interaction is required to identify open ports, service versions, or **misconfigurations** not visible through public records.

**Commands** Identify open ports and services on the target

```
nmap <TARGET_IP>
```

Execute service version detection to identify software like Apache or Nginx

```
nmap -sV <TARGET_IP>
```

Attempt to identify the target operating system

```
nmap -O <TARGET_IP>
```

Map network topology and hops to the target

```
traceroute <TARGET_IP>
```

Retrieve HTTP headers and service banners

```
curl -I <TARGET_IP>
```

**Tool comparison**

- Port Scanning
    - Nmap -> `nmap <TARGET_IP>` -> Default for comprehensive scanning and scripting
    - Masscan -> `masscan <TARGET_IP>` -> Alternative for high-speed scanning of large ranges
    - Unicornscan -> `unicornscan <TARGET_IP>` -> Specialized for specific scanning techniques
- Vulnerability Probing
    - Nikto -> `nikto -h <TARGET_IP>` -> Web-focused scanning for misconfigurations
    - Nessus -> Web UI -> Enterprise-grade vulnerability scanning
- Web Spidering
    - Burp Suite Spider -> GUI-based -> Integrated with proxy for manual analysis
    - OWASP ZAP Spider -> GUI-based -> Open-source alternative for mapping sites
    - Scrapy -> Custom scripts -> Preferred when highly customizable crawling behavior is required

**Gotchas** **Detection risk** is high; direct probes often trigger **IDS**, **firewalls**, or logging mechanisms.

> ⚠️ Gap: Automated vulnerability scanners send **exploit payloads** that are easily signatured and blocked by modern security solutions, leading to **silent failures** or IP blacklisting.