**Domain Information (Passive OSINT)**

Stay hidden — no direct contact with the target. You're a "visitor", not a scanner.

---

**SSL Certificates → Subdomains**

Check the company's SSL cert (often lists multiple domains), then use crt.sh:

```bash
# JSON output
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq .

# Unique subdomains only
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

---

**Resolve subdomains → IPs**

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4; done
```

---

**Shodan on discovered IPs**

```bash
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

Reveals open ports, services, versions, SSL config — without touching the target.

---

**DNS Records**

```bash
dig any inlanefreight.com
```

|Record|What it tells you|
|---|---|
|`A`|IP → subdomain mappings|
|`MX`|Mail provider (e.g. Google = Gmail)|
|`NS`|Name servers → often reveals hosting provider|
|`TXT`|Third-party services, SPF/DKIM, verification keys|
|`SOA`|Zone info|

**TXT records are goldmines** — they leak what services the company uses:

|Finding|Implication|
|---|---|
|Atlassian|Jira/Confluence → code repos, project data|
|Google Gmail|Possible open GDrive files|
|LogMeIn|Centralized remote access — high value target|
|Mailgun|API endpoints → test for IDOR, SSRF|
|Outlook/O365|OneDrive, Azure Blob/File (SMB!)|
|INWX|Hosting/domain registrar → possible username in MS= TXT record|
|SPF `ip4:` entries|Internal IPs leaking into public DNS|