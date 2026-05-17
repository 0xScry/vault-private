## Certificate Transparency Enumeration

Wordlist-based approaches are exhausted and a definitive record of certificates—including those for expired or unguessable subdomains—is required.

Filter crt.sh API results for specific patterns to isolate high-value subdomains

```
curl -s "https://crt.sh/?q=<DOMAIN>&output=json" | jq -r '.[] | select(.name_value | contains("<KEYWORD>")) | .name_value' | sort -u
```

- crt.sh
    
    - `https://crt.sh`
    - Use for rapid, registration-free discovery of subdomains and SAN entries.
- Censys
    
    - `https://censys.io`
    - Use for advanced filtering by IP or certificate attributes and finding related misconfigured hosts.
- **Stale infrastructure** found through expired certificates may host **outdated software** vulnerable to exploitation.
    
- **Manual filtering** via the crt.sh web interface is inefficient for large-scale analysis compared to API automation.
    

> ⚠️ Gap: CT log entries confirm name issuance but do not verify current DNS resolution; targeting these subdomains without secondary resolution checks can result in attacking decommissioned or reassigned infrastructure.