1. Scrutinize main website text and SSL certificates to identify initial service structures and secondary domains.
2. Query Certificate Transparency logs via crt.sh to expand the subdomain list.
3. Resolve subdomains to IPs and filter out third-party hosted infrastructure to define the actionable attack surface.
4. Pivot target-owned IPs to Shodan to identify open ports and service versions without direct interaction.
5. Execute DNS ANY queries to find TXT records revealing third-party SaaS integrations like Atlassian, O365, or Mailgun.

---

## Subdomain Discovery via Certificate Transparency

When to use: Target domain is known and you need to find subdomains or sibling domains via public logs.

Fetch raw JSON data for a domain from crt.sh

```
curl -s https://crt.sh/?q=<DOMAIN>&output=json | jq .
```

Parse crt.sh JSON output into a unique list of subdomains

```
curl -s https://crt.sh/?q=<DOMAIN>&output=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\n/,"\n");}1;' | sort -u
```

- Gotcha: **Results will differ** between the web interface and command line tools depending on how the database stores and retrieves new entries.

## Infrastructure Validation

When to use: A subdomain list exists and you must separate target-owned assets from third-party providers.

Filter a subdomain list to identify IPs associated with the target domain

```
for i in $(cat <FILE_PATH>);do host $i | grep "has address" | grep <DOMAIN> | cut -d" " -f1,4;done
```

- Gotcha: **Permission failure** occurs if you test hosts belonging to third-party providers without their specific authorization.

## Passive Service Mapping

When to use: Validated target IPs are identified and you need port, protocol, and banner information without making direct connections.

Extract IP addresses from a validated subdomain list for further scanning

```
for i in $(cat <FILE_PATH>);do host $i | grep "has address" | grep <DOMAIN> | cut -d" " -f4 >> ip-addresses.txt;done
```

Query Shodan for host information, open ports, and service banners using a list of IPs

```
for i in $(cat ip-addresses.txt);do shodan host $i;done
```

- Gotcha: **Stale data** may be present as Shodan relies on its most recent crawl rather than real-time state.

## DNS and SaaS Footprinting

When to use: Mapping the company's reliance on external platforms and identifying potential API or remote access vectors.

Retrieve all available DNS records for a domain

```
dig any <DOMAIN>
```

- Edge cases: Look for **MS** or **verification-code** values in TXT records which often reveal the username or ID used for management platforms.
- Gotcha: **Administrative takeover** is possible if you obtain credentials for centralized platforms like LogMeIn which manage remote access across the infrastructure.