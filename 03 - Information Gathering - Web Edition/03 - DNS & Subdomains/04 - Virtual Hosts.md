## Manual Virtual Host Mapping

When to use: Targeted virtual host lacks a **DNS record** or is restricted to internal access.

Commands Map a domain name to a known IP address manually by editing the local hosts file to **bypass DNS resolution**

```
<TARGET_IP> <DOMAIN>
```

Gotchas **Missing DNS records** will prevent the browser or tools from reaching the target unless the local hosts file is updated.

---

## Automated Virtual Host Discovery

When to use: Fuzzing the **Host header** to identify public or non-public subdomains and distinct domains sharing a single IP address.

Commands Standard VHost brute-forcing using Gobuster with mandatory domain appending for correct hostname construction

```
gobuster vhost -u http://<DOMAIN> -w <FILE_PATH> --append-domain
```

Tool comparison

- Gobuster -> `gobuster vhost -u <URL> -w <WORDLIST>` -> prefer for dedicated VHost enumeration mode and speed.
- Feroxbuster -> Rust-based implementation -> prefer when requiring **recursion**, wildcard discovery, or flexible filtering.
- ffuf -> Web fuzzer -> prefer for highly **customizable wordlist input** and advanced response filtering.

Edge cases Newer versions of Gobuster require the `--append-domain` flag to function correctly; older versions may append the base domain by default or lack the flag entirely.

Gotchas **High traffic generation** during discovery frequently triggers **IDS/WAF detection**. **Failure to use --append-domain** in recent Gobuster versions prevents the tool from constructing the full virtual hostnames required for accurate enumeration.

---

## Name-Based Virtual Host Configuration

When to use: Analyzing server-side routing where the **Host header** acts as a switch to determine which **DocumentRoot** is served.

Dangerous / misconfigured settings

- Multiple distinct domains (e.g., .com, .org, .net) hosted on a single IP using the same port.
- Apache `VirtualHost` blocks where `ServerName` is the only differentiator for sensitive content.

Gotchas **Incorrect Host headers** will cause the web server to serve the default site or fail to route to the intended application content.