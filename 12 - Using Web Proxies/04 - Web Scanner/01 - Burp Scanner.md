## Target Scope Management

Application has **logout functions** or **out-of-scope traffic** is bloating history; target site map populated.

Define specific inclusion/exclusion criteria to focus tool resources

```
Target > Scope > Use advanced scope control
```

Add target to inclusion list to enable filtered scanning

```
Right-click <TARGET_IP> in Site map -> Add to scope
```

Exclude endpoints like logout functions to prevent **session termination**

```
Right-click <FILE_PATH> -> Remove from scope
```

> ⚠️ Gap: Burp Scanner functionality is **Pro-only**; standard menu options for active/passive scanning do not exist in Community Edition.

## Site Mapping and Crawling

Map of website structure is empty or incomplete; target URL is in scope.

Initialize mapping using the fastest built-in strategy for rapid discovery

```
Dashboard -> New Scan -> Crawl -> Scan configuration -> Select from library -> Crawl strategy - fastest
```

Execute a targeted crawl from a specific captured request

```
Right-click request in Proxy History -> Scan -> Crawl
```

- **Dangerous / misconfigured settings**
    
    - **Unauthenticated crawling** misses restricted application areas.
    - **Missing scope limits** cause the crawler to follow links to external domains.
- **Edge cases**
    
    - Standard crawl **cannot identify unreferenced pages** or hidden directories; necessitates manual addition of findings from dirbuster or ffuf to the site map.
- **Gotchas** **Session timeout** occurs if crawling triggers logout forms without exclusion rules.
    

## Passive Vulnerability Assessment

Quick analysis required for **DOM-based XSS** or **missing HTML tags**; request/response already captured in history.

Analyze existing traffic for vulnerabilities without sending new requests

```
Right-click <TARGET_IP> in Site map -> Passively scan this target
```

- **Gotchas** **Low confidence** ratings indicate potential false positives since issues are not verified with payloads.

---

## Active Vulnerability Scanning

Site map built and testing for **High severity** flaws like **OS command injection** required; target supports request modification.

Combine site mapping and vulnerability testing in one operation

```
Dashboard -> New Scan -> Crawl and Audit
```

Restrict scan to **critical issues only** to reduce execution time and noise

```
Audit configuration -> Select from library -> Audit checks - critical issues only
```

Audit a specific parameter or request discovered during manual proxying

```
Right-click request in Proxy History -> Do active scan
```

- **Edge cases**
    
    - Authenticated scanning requires recorded login steps or provided credentials to reach **server-side** logic behind auth walls.
- **Gotchas** **Scan timeout** or excessive duration occurs when using default configurations on large targets without limiting audit checks.
    

## Vulnerability Reporting

Exporting proof-of-concept data and remediation details for appendix material.

Generate a customized HTML/XML report for identified host issues

```
Target > Site map -> Right-click <TARGET_IP> -> Issue -> Report issues for this host
```

- **Dangerous / misconfigured settings**
    
    - Submitting raw Burp reports as final deliverables without manual verification or professional formatting.
- **Gotchas** **Firm confidence** issues require manual re-testing of the sent request/response to confirm exploitability.