1. Check local resolution overrides in `<FILE_PATH>` to ensure traffic isn't being diverted.
2. Query the **SOA** and **NS** records to identify the administrative head and authoritative servers for the target zone.
3. Perform a manual sweep of common records (A, MX, TXT) using `dig` or `host`.
4. Use `dig -x` on identified IP ranges to find associated hostnames via **PTR** records.
5. Launch automated enumeration tools if subdomains remain hidden or the target surface is large.

---

## Local Host Resolution

Manual mapping to bypass the DNS resolution process for development or redirection.

Modify the local text file to force resolution to a specific IP

```
cat <FILE_PATH>
```

Update the file with the target mapping

```
<TARGET_IP>    <DOMAIN>
```

- Linux/MacOS: `/etc/hosts`
- Windows: `C:\Windows\System32\drivers\etc\hosts`

**Immediate effect** occurs upon saving the file without requiring a system restart.

## Manual Record Querying

Standard queries to pull IPv4, IPv6, mail servers, and infrastructure metadata.

### dig

Versatile querying for deep analysis of DNS records and resolution paths.

Standard A record lookup

```
dig <DOMAIN> A
```

Query a specific name server for records

```
dig @<TARGET_IP> <DOMAIN>
```

Full resolution path tracing from root to authoritative server

```
dig +trace <DOMAIN>
```

Retrieve only the result string to pipe into other tools

```
dig +short <DOMAIN>
```

Filtered output showing only the answer section

```
dig +noall +answer <DOMAIN>
```

- `dig` -> `dig <DOMAIN> <TYPE>` -> Prefer for detailed troubleshooting and full resolution tracing.
- `nslookup` -> `nslookup <DOMAIN>` -> Prefer for quick A/AAAA or MX checks.
- `host` -> `host <DOMAIN>` -> Prefer for concise, one-line output for scripts.

**RFC 8482** causes many DNS servers to ignore **ANY** queries to prevent abuse, potentially returning no records even if they exist.

### host

Streamlined querying for rapid checks.

Quick resolution of IPv4 and mail records

```
host <DOMAIN>
```

## Reverse DNS Discovery

Identifying hostnames associated with a known IP address.

Perform a reverse lookup for the PTR record

```
dig -x <TARGET_IP>
```

**Missing PTR records** will cause the reverse lookup to return no hostname despite the IP being active.

## Automated DNS Enumeration

Scaling discovery through dictionary attacks and brute-forcing when manual queries are insufficient.

### dnsenum

Automated discovery including dictionary attacks and brute-forcing.

Run comprehensive enumeration and attempt zone transfers

```
dnsenum <DOMAIN>
```

### fierce

Recursive search functionality with built-in wildcard detection.

Scan for subdomains and identify potential targets

```
fierce --domain <DOMAIN>
```

### dnsrecon

Combines multiple techniques and supports diverse output formats.

Execute general enumeration and record gathering

```
dnsrecon -d <DOMAIN>
```

### theHarvester

OSINT-based discovery for non-technical data associated with DNS.

Gather email addresses and employee info from DNS and external sources

```
theHarvester -d <DOMAIN> -b all
```

**Rate limiting** by target servers can lead to blocked queries or temporary IP bans during aggressive scanning.