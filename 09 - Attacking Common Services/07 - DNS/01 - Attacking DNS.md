# DNS Enumeration and Attacks

The **Domain Name System (DNS)** translates domain names into numerical IP addresses. While it primarily uses **UDP/53**, it falls back to **TCP/53** for large packets or reliable data transmission during zone transfers. Attacks against DNS allow attackers to map an organization's infrastructure, services, and third-party providers.

## Initial Enumeration

Use this technique to identify the DNS server version and run default scripts to find initial vulnerabilities or configuration details.

|Command|Description|
|:--|:--|
|`nmap -p53 -Pn -sV -sC <TARGET_IP>`|Scans for DNS service version and runs default scripts.|

## DNS Zone Transfer (AXFR)

**When to use:** Use when a DNS server is suspected of being misconfigured to allow unauthenticated requests for zone information. **Why it matters:** A successful transfer **dumps the entire DNS namespace**, revealing internal hosts and significantly increasing the target's attack surface.

### Operational Workflow

1. **Identify Vulnerable Servers:** Attempt to dump the zone file using the `AXFR` query type.
2. **Automated Discovery:** Use specialized tools to find all name servers for a domain and automatically test each for zone transfer vulnerabilities.

|Command|Tool|Purpose|
|:--|:--|:--|
|`dig AXFR @<NAMESERVER_IP> <DOMAIN>`|`dig`|Manually requests a full zone transfer from a specific nameserver.|
|`fierce --domain <DOMAIN>`|`Fierce`|Enumerates all DNS servers for a root domain and scans them for zone transfers.|

## Subdomain Enumeration & Takeover

**When to use:** Use when mapping the external infrastructure of a target or when searching for orphaned third-party service links. **Why it matters:** Identifying a **subdomain takeover** allows an attacker to gain complete control over a subdomain (e.g., hosting malicious content or phishing) by claiming an expired resource that a **CNAME record** still points to.

### Operational Workflow

1. **Scrape Subdomains:** Use open-source intelligence to find existing subdomains.
2. **Internal Brute-forcing:** If performing an internal pentest without internet access, use a tool with self-defined resolvers.
3. **Analyze CNAME Records:** Check if subdomains point to third-party services (e.g., AWS, GitHub).
4. **Verify Vulnerability:** Check for errors like `NoSuchBucket`, which indicate the external resource is available for registration.

### Command Reference

|Command|Tool|Purpose|
|:--|:--|:--|
|`./subfinder -d <DOMAIN> -v`|`Subfinder`|Scrapes subdomains from open sources like DNSdumpster.|
|`subbrute.py <DOMAIN> -s <WORDLIST> -r <RESOLVERS>`|`Subbrute`|Performs pure DNS brute-forcing; useful for internal pivoting.|
|`host <SUBDOMAIN>`|`host`|Retrieves the **CNAME** record to identify where a subdomain points.|

### Misconfiguration Table

|Setting/Condition|Impact|
|:--|:--|
|**Dangling CNAME**|Points to an expired domain or service; allows **Subdomain Takeover**.|
|**Unrestricted AXFR**|Allows any IP to download the full DNS database without authentication.|

## DNS Spoofing (Cache Poisoning)

**When to use:** Use during **Man-in-the-Middle (MITM)** attacks on a local network to redirect traffic. **Why it matters:** This attack alters legitimate DNS records with false information, forcing victims to visit a fraudulent site instead of the intended destination.

### Operational Workflow

1. **Configure Spoofing Map:** Edit the tool configuration file to map the legitimate domain to the `<ATTACK_IP>`.
2. **Identify Targets:** Scan the local network for the victim host and the default gateway.
3. **Execute Poisoning:** Activate the spoofing plugin to send fake DNS responses to the victim.

|Step|Action/Command|
|:--|:--|
|1|Edit `/etc/ettercap/etter.dns` and add: `<DOMAIN> A <ATTACK_IP>`.|
|2|Open `Ettercap` and navigate to **Hosts > Scan for Hosts**.|
|3|Add `<TARGET_IP>` to **Target 1** and Gateway IP to **Target 2**.|
|4|Navigate to **Plugins > Manage Plugins** and activate `dns_spoof`.|

**Attack Implication:** Once active, any requests (e.g., web browsing or pings) from the victim to the mapped domain will resolve to the `<ATTACK_IP>`.