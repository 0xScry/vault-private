# Virtual Host Discovery and Enumeration

### Virtual Hosting Fundamentals

Web servers (Apache, Nginx, IIS) use **virtual hosting** to host multiple websites or applications on a single server sharing the same IP address. The server distinguishes between these sites by leveraging the **HTTP Host header** included in every browser request. The Host header acts as a switch, telling the web server which specific content or **DocumentRoot** to serve.

### VHost Fuzzing Methodology

**VHost fuzzing** is used to discover public and non-public subdomains or virtual hosts by testing various hostnames against a known IP address.

- **When to use:** Use this technique when you suspect a server hosts multiple sites that are not listed in public **DNS records**.
- **Why it matters:** Many subdomains are only accessible internally or through specific configurations and will not appear in standard DNS lookups.
- **Manual Bypass:** If a virtual host lacks a DNS record, you can manually map the domain name to the IP address in your local **hosts file** to bypass DNS resolution.

### Virtual Host Discovery Tools

Specialized tools automate the process of probing a target server by systematically testing different Host headers.

|Tool|Description|Key Features|
|:--|:--|:--|
|**Gobuster**|Multi-purpose tool for brute-forcing.|Fast; supports multiple HTTP methods and custom wordlists.|
|**Feroxbuster**|Rust-based implementation.|High speed, supports recursion and wildcard discovery.|
|**ffuf**|Flexible web fuzzer.|Highly customizable wordlist input and filtering.|

### Operational Workflow: Gobuster VHost Discovery

To identify valid virtual hosts, Gobuster sends HTTP requests with varying Host headers and analyzes the server responses.

1. **Prepare the target URL:** Identify the base IP or domain and the port.
2. **Select a wordlist:** Use a specialized DNS or subdomain wordlist.
3. **Execute the scan:** Use the `vhost` mode to test the headers.
4. **Analyze Results:** Note valid hosts (e.g., Status 200) for further investigation.

#### Command Reference

|Parameter|Description|
|:--|:--|
|`vhost`|Sets Gobuster to virtual host enumeration mode.|
|`-u`|Specifies the target URL.|
|`-w`|Specifies the path to the wordlist.|
|`--append-domain`|Appends the base domain to each wordlist entry (required in newer versions).|

**Basic VHost Enumeration:**

```
gobuster vhost -u http://<TARGET_IP> -w <PATH_TO_WORDLIST> --append-domain
```

**Enumeration on Custom Port:**

```
gobuster vhost -u http://<DOMAIN>:<PORT> -w <PATH_TO_WORDLIST> --append-domain
```

### Attack Implications and Risks

- **Discovery:** Successful fuzzing unlocks access to internal applications, staging environments, or hidden administrative panels.
- **Detection Risk:** This process generates significant network traffic and is likely to be detected by **Intrusion Detection Systems (IDS)** or **Web Application Firewalls (WAF)**.

### Server-Side Configuration (Apache Example)

Virtual hosts can be configured as subdomains or entirely different domains on the same infrastructure.

|Configuration Component|Purpose|
|:--|:--|
|`<VirtualHost *:80>`|Defines the IP and port for the virtual host.|
|`ServerName`|The domain name the server looks for in the Host header.|
|`DocumentRoot`|The directory containing the files for that specific domain.|

**Example Name-Based Configuration:**

```
<VirtualHost *:80>
    ServerName www.<DOMAIN_ONE>.com
    DocumentRoot /var/www/<DOMAIN_ONE>
</VirtualHost>

<VirtualHost *:80>
    ServerName www.<DOMAIN_TWO>.org
    DocumentRoot /var/www/<DOMAIN_TWO>
</VirtualHost>
```