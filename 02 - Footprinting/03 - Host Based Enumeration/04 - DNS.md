# DNS Footprinting and Analysis

The **Domain Name System (DNS)** resolves computer names into IP addresses. It is a distributed system with no central database, utilizing various server types and record formats to manage global and local naming.

## DNS Infrastructure Reference

### Server Types

|Server Type|Description|
|:--|:--|
|**Root Server**|Handles Top-Level Domains (TLD) and acts as the central interface linking domains to IPs.|
|**Authoritative**|Holds binding authority for a specific zone and only answers queries within its responsibility.|
|**Non-authoritative**|Collects information via recursive or iterative queries; not responsible for the zone.|
|**Caching DNS**|Temporarily stores information from other servers for a duration set by the authoritative server.|
|**Forwarding**|Forwards queries to another designated DNS server.|
|**Resolver**|Performs local name resolution on a computer or router.|

### DNS Record Types

|Record|Function|
|:--|:--|
|**A**|Returns an **IPv4** address.|
|**AAAA**|Returns an **IPv6** address.|
|**MX**|Identifies responsible **mail servers**.|
|**NS**|Lists the **nameservers** of the domain.|
|**TXT**|Holds various info (SSL validation, SPF/DMARC for mail security).|
|**CNAME**|An **alias** for another domain name.|
|**PTR**|Used for **reverse lookups** (IP to domain).|
|**SOA**|Provides zone information and administrative contact email.|

---

## Dangerous Misconfigurations

Misconfigurations often occur when functionality is prioritized over security during troubleshooting.

|Option|Security Risk|
|:--|:--|
|**allow-query**|Defines which hosts can send requests; if too broad, anyone can probe the server.|
|**allow-recursion**|Defines hosts allowed to send recursive requests.|
|**allow-transfer**|If set to `any` or an overly broad subnet, it allows **unauthorized zone transfers**.|

---

## Operational Workflow: DNS Footprinting

### 1. Identify Target Nameservers

Querying for the **NS record** is the first step. Finding additional nameservers allows you to query them individually, as they may have different configurations or hold different zones.

```
dig ns <DOMAIN> @<TARGET_IP>
```

### 2. Determine Software Version

Identify the version of the BIND server using a **CHAOS** class query. This is useful for identifying specific vulnerabilities associated with that version.

```
dig CH TXT version.bind <TARGET_IP>
```

### 3. Enumerate Records (ANY Query)

Request all available records the server is willing to disclose. Note that this may not return the entire zone, only what the server is configured to show.

```
dig any <DOMAIN> @<TARGET_IP>
```

### 4. Attempt Zone Transfer (AXFR)

**When to use:** Use this early in the assessment to check if the `allow-transfer` setting is misconfigured. **Why it matters:** A successful AXFR reveals the **entire zone file**, including internal IP addresses, hostnames, and roles (e.g., domain controllers, mail servers), effectively providing a map of the internal network.

```
dig axfr <DOMAIN> @<TARGET_IP>
```

_If targeting a potential internal subdomain discovered during the process:_

```
dig axfr <SUBDOMAIN>.<DOMAIN> @<TARGET_IP>
```

### 5. Subdomain Brute Forcing

**When to use:** If zone transfers are blocked, use brute forcing to identify active hostnames. **Attack Implication:** This unlocks further targets (A records) that are not publicly indexed.

**Option A: Bash Loop** Use this for a quick, lightweight check using a standard wordlist.

```
for sub in $(cat /path/to/wordlist.txt); do dig $sub.<DOMAIN> @<TARGET_IP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done
```

**Option B: DNSenum** Use for automated, comprehensive enumeration, including version checks and brute forcing in one step.

```
dnsenum --dnsserver <TARGET_IP> --enum -p 0 -s 0 -o subdomains.txt -f /path/to/wordlist.txt <DOMAIN>
```

---

## BIND9 Configuration Reference

The BIND9 configuration (`named.conf`) is split into **global options** and **zone options**. Zone-specific options always take precedence over global ones.

- **Zone Files:** Must contain exactly one **SOA** and at least one **NS** record.
- **Reverse Lookup Files:** Map the last octet of an IP to an FQDN using **PTR** records.
- **Failure Condition:** A syntax error in a zone file causes the server to return a `SERVFAIL` error, treating the zone as if it does not exist.