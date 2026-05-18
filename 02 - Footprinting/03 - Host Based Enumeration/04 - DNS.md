## DNS Record Enumeration

Target domain is identified and infrastructure mapping is required to find mail servers, nameservers, or service validation strings.

Query nameservers to identify secondary targets for further record interrogation

```
dig ns <DOMAIN> @<TARGET_IP>
```

Request all disclosed records to find TXT/SPF data or SOA information

```
dig any <DOMAIN> @<TARGET_IP>
```

Extract SOA record to identify the administrative contact and zone serial number

```
dig soa <DOMAIN> @<TARGET_IP>
```

Perform reverse lookup to resolve an IP address to a hostname

```
dig -x <TARGET_IP> @<TARGET_IP>
```

> ⚠️ Gap: Reverse lookups will fail if a dedicated **reverse lookup zone file** (PTR records) is not configured on the DNS server for the specific subnet.

**NOERROR** status in a response with an empty answer section confirms the record type does not exist for that specific host.

## Service Identification

Investigating BIND-specific vulnerabilities that require precise version strings for exploit selection.

Query version using CHAOS class and TXT type

```
dig CH TXT version.bind <TARGET_IP>
```

**Entry must exist** on the server; if the administrator has not configured the version.bind record, the query returns no data.

## Zone Transfer (AXFR)

Nameservers are identified and require testing for insecure synchronization settings that leak the entire zone file.

Attempt full zone dump over TCP 53

```
dig axfr <DOMAIN> @<TARGET_IP>
```

Attempt transfer for internal subdomains discovered in the primary zone dump

```
dig axfr internal.<DOMAIN> @<TARGET_IP>
```

- `allow-transfer` set to **any** or a broad testing subnet allows unauthorized users to dump the entire zone file.

**AXFR record query failed** occurs when the nameserver is unreachable or transfers are restricted by IP ACLs or a secret **rndc-key**.

## Subdomain Discovery

Zone transfers are blocked or restricted, necessitating wordlist-based discovery of hidden hostnames.

Automated enumeration including automated zone transfer attempts and scraping

```
dnsenum --dnsserver <TARGET_IP> --enum -p 0 -s 0 -o <FILE_PATH> -f <FILE_PATH> <DOMAIN>
```

Manual Bash loop for surgical record extraction and custom filtering

```
for sub in $(cat <FILE_PATH>); do dig $sub.<DOMAIN> @<TARGET_IP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done
```

- `dnsenum` -> `dnsenum --dnsserver <TARGET_IP> --enum <DOMAIN>` -> prefer for comprehensive, multi-threaded automated scanning.
- Bash Loop -> `for sub in $(cat <WORDLIST>); do dig $sub.<DOMAIN> @<TARGET_IP>; done` -> prefer for stealthier, manual interrogation of specific targets.

**SERVFAIL** indicates a syntax error in the server's zone file, causing the server to treat the zone as non-existent.