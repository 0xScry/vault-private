1. Need specific metadata (Headers, WHOIS, SSL) -> use targeted flags
2. Need infrastructure mapping (DNS, Subdomains, Port scan) -> use infrastructure flags
3. Need content discovery (Directory search, Crawl, Wayback) -> use discovery flags
4. Need full automated surface analysis -> use `--full`
5. Default wordlist or threading insufficient -> specify `-w` and `-dt` or `-pt`

---

## FinalRecon Setup

Installation required before initial execution.,

Clone repository and install dependencies

```
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
```

## Targeted Information Gathering

Gathering specific metadata when manual reconnaissance is slow or prone to error.,

Execute header and WHOIS lookup for a specific target

```
./finalrecon.py --headers --whois --url <URL>
```

**Gotchas**

- **Redirects are disabled** by default; use `-r` to follow them.
- **SSL verification is active** by default; use `-s` to toggle if encountering self-signed certificates.

## Infrastructure Enumeration

Mapping the external attack surface via DNS, subdomains, and certificates.,

Perform DNS and subdomain enumeration using custom DNS servers

```
./finalrecon.py --dns --sub -d <DNS_SERVER> --url <URL>
```

Fast port scan and SSL certificate inspection

```
./finalrecon.py --ps --sslinfo --url <URL>
```

**Dangerous / misconfigured settings**

- Default port scan threads at 50; **aggressive scanning** may trigger security alerts.
- Default SSL port is 443; **non-standard ports** require `-sp <PORT>`.

## Content and Path Discovery

Identifying hidden directories, files, and historical URL data.,

Directory search with specific extensions and custom wordlist

```
./finalrecon.py --dir -w <FILE_PATH> -e <EXTENSIONS> --url <URL>
```

Crawl target and retrieve Wayback Machine URLs

```
./finalrecon.py --crawl --wayback --url <URL>
```

**Edge cases**

- Adjust directory threads with `-dt` if the target server is unstable or high-latency.

## Comprehensive Reconnaissance

Automating all available modules for a complete initial overview.,

Run all reconnaissance modules against the target

```
./finalrecon.py --full --url <URL>
```

Modify export directory and hide banner for cleaner output

```
./finalrecon.py --full -nb -cd <FILE_PATH> --url <URL>
```

**Gotchas**

- **API keys are required** for certain external data sources; use `-k shodan@<HASH>` to provide them.