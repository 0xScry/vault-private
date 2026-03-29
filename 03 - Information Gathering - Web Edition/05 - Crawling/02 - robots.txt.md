# Robots.txt Reconnaissance

### Overview

**Robots.txt** is a plain text file located in the **root directory** of a website that follows the **Robots Exclusion Standard**. It serves as a set of instructions (directives) for web crawlers, specifying which parts of a site they are permitted to access.

### Reconnaissance Methodology

Use this technique during the **initial information gathering** phase to identify the internal structure of a web application and locate hidden directories.

1. **Access the file**: Navigate to the root of the target domain or IP.
    
    ```
    http://<TARGET_IP>/robots.txt
    ```
    
2. **Identify User-Agents**: Check if directives apply to all bots (`User-agent: *`) or specific entities (e.g., `User-agent: Googlebot`).
3. **Map Disallowed Paths**: Document all directories marked with `Disallow` to find areas the owner wants to hide from public search engines.
4. **Locate Sitemaps**: Use the `Sitemap` directive to find the XML map of the site for efficient content discovery.

### Directive Reference

|Directive|Description|Attack Implication|
|:--|:--|:--|
|`Disallow`|Paths/patterns bots should not crawl.|**Unlocks potential targets** like admin panels (`/admin/`) or sensitive data (`/private/`).|
|`Allow`|Permits access to specific paths within disallowed zones.|Identifies accessible files or entry points within restricted directories.|
|`Crawl-delay`|Wait time (seconds) between bot requests.|Indicates potential server resource limitations or rate-limiting configurations.|
|`Sitemap`|URL to the site's XML sitemap.|Provides a comprehensive list of URLs to further expand the attack surface.|

### Analysis and Decision Logic

- **Identify High-Value Targets**: If `Disallow: /admin/` is present, it confirms the existence of an administrative interface.
- **Identify Hidden Content**: Files or directories listed under `Disallow` often contain sensitive information that is not intended for public viewing, making them primary targets for further enumeration.
- **Enforcement Limitation**: While legitimate bots respect these rules, the file is **not strictly enforceable**; a security professional can ignore these instructions to access restricted areas manually.

### Example Analysis

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Sitemap: https://<DOMAIN>/sitemap.xml
```

- **Inference**: The target likely hosts an administrative login or panel at `http://<TARGET_IP>/admin/` and potentially sensitive files in `http://<TARGET_IP>/private/`.