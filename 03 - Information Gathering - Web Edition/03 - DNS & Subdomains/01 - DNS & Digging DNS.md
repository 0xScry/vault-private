### DNS Fundamentals for Reconnaissance

DNS acts as a translation layer between human-readable domain names and numerical IP addresses. For a penetration tester, DNS is a **critical component of target infrastructure** that can be leveraged to uncover vulnerabilities and gain entry points.

#### Local Name Resolution: The Hosts File

The **hosts file** provides a manual method of domain resolution that **bypasses the standard DNS process**, allowing for local overrides.

- **When to use:** Use for development, troubleshooting, or to manually point a domain to a specific IP during a pentest (e.g., when a web server is not yet in public DNS).
- **Format:** `<IP_ADDRESS> <HOSTNAME> [ALIAS]`.

|Operating System|File Path|
|:--|:--|
|**Windows**|`C:\Windows\System32\drivers\etc\hosts`|
|**Linux / MacOS**|`/etc/hosts`|

---

#### DNS Infrastructure and Record Types

DNS data is organized into **zones**, which are managed containers for specific domain namespaces. The **zone file** defines the resource records within that container.

|Record Type|Description|Reconnaissance Value|
|:--|:--|:--|
|**A**|Maps hostname to **IPv4** address.|Identifies target web servers and infrastructure.|
|**AAAA**|Maps hostname to **IPv6** address.|Identifies IPv6-enabled infrastructure.|
|**CNAME**|Creates an **alias** for a hostname.|Can reveal third-party hosting or internal naming conventions.|
|**MX**|Identifies **mail servers**.|Essential for identifying email infrastructure and potential phishing targets.|
|**NS**|Identifies **authoritative name servers**.|Determines which server holds the actual IP data for a domain.|
|**TXT**|Stores text (often **SPF** or verification).|Can reveal security policies or third-party service integration.|
|**SOA**|**Start of Authority** administrative info.|Provides admin email and primary name server details.|
|**SRV**|Defines host and **port** for specific services.|Reveals service locations and their associated ports.|
|**PTR**|Maps IP address to hostname (**Reverse Lookup**).|Used to identify the host associated with a known IP.|

---

#### DNS Reconnaissance Tools

Reconnaissance involves querying DNS servers to extract infrastructure data and discover subdomains.

|Tool|Key Use Case|
|:--|:--|
|**dig**|Versatile, detailed DNS lookups and **zone transfers**.|
|**nslookup**|Simple queries for A, AAAA, and MX records.|
|**host**|Streamlined tool for quick A, AAAA, and MX checks.|
|**dnsenum**|**Automated enumeration**, dictionary attacks, and brute-forcing.|
|**fierce**|Subdomain enumeration with recursive search and wildcard detection.|
|**dnsrecon**|Comprehensive enumeration using multiple techniques.|
|**theHarvester**|OSINT tool for gathering emails and employee info via DNS.|

---

#### Operational Workflow: Querying with `dig`

The **Domain Information Groper (dig)** is the primary utility for manual DNS analysis due to its flexibility and detailed output.

**1. General Domain Enumeration** Query specific records to build a profile of the target's external infrastructure.

|Goal|Command|
|:--|:--|
|**Default Lookup** (A Record)|`dig <DOMAIN>`|
|**IPv4 Address**|`dig <DOMAIN> A`|
|**IPv6 Address**|`dig <DOMAIN> AAAA`|
|**Mail Servers**|`dig <DOMAIN> MX`|
|**Name Servers**|`dig <DOMAIN> NS`|
|**Text Records**|`dig <DOMAIN> TXT`|
|**All Available Records**|`dig <DOMAIN> ANY`|

**2. Targeted Server Queries** Use when you need to query a specific DNS server (e.g., a discovered internal DNS server) rather than the default resolver.

- **Command:** `dig @<TARGET_IP> <DOMAIN>`

**3. Reverse IP Mapping** Use to find the hostname associated with a specific IP address.

- **Command:** `dig -x <TARGET_IP>`

**4. Trace Resolution Path** Use to see the full path of DNS resolution from root servers down to the authoritative server.

- **Command:** `dig +trace <DOMAIN>`

**5. Clean Output for Scripting** Use these flags to remove header/footer clutter and extract only the relevant data.

|Flag|Effect|
|:--|:--|
|`+short`|Provides only the answer (e.g., the IP address).|
|`+noall +answer`|Displays only the answer section of the output.|

---

#### Operational Security (OPSEC) and Limitations

- **Detection:** Some servers detect and **block excessive queries**. Always respect rate limits.
- **ANY Queries:** Many modern servers ignore `ANY` queries to prevent abuse (RFC 8482).
- **Permissions:** Always obtain permission before performing extensive reconnaissance.