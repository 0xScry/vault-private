## Banner Grabbing

Need to extract server software versions and redirection logic from HTTP headers without downloading page content.

Fetch headers to identify web server and OS.

```
curl -I <DOMAIN>
```

Follow redirections manually to identify CMS-specific headers like `X-Redirect-By`.

```
curl -I https://<DOMAIN>
```

**Gotchas** **Redirects hide technologies** if only the initial 301/302 response is inspected without following the chain to the final 200 OK destination.

## WAF Detection

Identify active filtering solutions that may block or alert on reconnaissance probes.

Execute automated signature-based detection for Web Application Firewalls.

```
wafw00f <DOMAIN>
```

**Gotchas** **Active WAF presence** will filter or block subsequent automated scans, requiring a shift to evasive techniques.

## Software Identification and Vulnerability Scanning

Identifying outdated software, insecure files, and CMS entry points through automated fingerprinting modules.

Run Nikto limited to software identification to minimize noise.

```
nikto -h <DOMAIN> -Tuning b
```

- Wappalyzer: Browser extension → prefer for quick visual profiling while browsing.
    
- BuiltWith: Web service → prefer for deep historical technology stack reports.
    
- WhatWeb: CLI tool → prefer for automated scanning via vast signature databases.
    
- Nmap: Network scanner → prefer when combining port scanning with NSE-based fingerprinting.
    
- Netcraft: Web service → prefer for identifying hosting providers and security posture.
    
- Missing `Strict-Transport-Security` header on TLS sites.
    
- Missing `X-Content-Type-Options` header allowing MIME-type sniffing.
    
- `Content-Encoding` set to `deflate` enabling BREACH attacks.
    
- Exposed `license.txt` or `wp-json` paths confirming software presence.
    
- Cookies set without the `httponly` flag.
    

**Gotchas** **Junk HTTP methods** returning valid responses can cause false positives in automated scanners.