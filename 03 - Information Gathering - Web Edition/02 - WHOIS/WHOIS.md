1. Query primary domains to identify the **Registrant Organization** for cross-referencing parent companies or subsidiaries.
2. Map **name servers** to identify where the target manages external DNS.
3. Extract **IP address blocks** and **autonomous systems** to define the organization's network boundaries.
4. Pivot on identified infrastructure patterns, such as hosting providers or specific IP ranges, to discover secondary domains.

---

## Utility Installation

When to use: Tool is missing from the local environment.

Update repositories and install the client

```
sudo apt update && sudo apt install whois -y
```

## Domain and Registrar Profiling

When to use: Establishing the baseline for a target domain, registrar, and administrative contact data.

Query the WHOIS database for record details

```
whois <DOMAIN>
```

**Gotchas** **Incomplete attribution** occurs when records fail to identify individual employees or internal vulnerabilities, requiring enrichment through secondary recon.

## Network Boundary Mapping

When to use: Moving beyond domains to identify owned **IP address blocks** and **autonomous systems** linked to the target.

Query for IP block and network resource ownership

```
whois <TARGET_IP>
```

**Gotchas** **Database latency** results in stale data; verify the "Last update of whois database" field to ensure information is current.

## Organizational Asset Correlation

When to use: Using a confirmed **Registrant Organization** or infrastructure pattern to find additional assets or profile TTPs.

Verify the organization name and registration dates

```
whois <DOMAIN>
```

**Edge cases**

- **Hidden registrant** info and recent creation dates typically indicate phishing or burner infrastructure rather than corporate assets.
- **Suspicious hosting** or "bulletproof" servers identified via WHOIS indicate high-risk command-and-control (C2) infrastructure.

**Gotchas** **Redacting registrant** data via privacy services can mask organizational ownership, necessitating pivots on **name servers** or IP ranges to maintain the map.