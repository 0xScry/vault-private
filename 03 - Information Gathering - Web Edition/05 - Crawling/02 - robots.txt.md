## Robots.txt Analysis

When to use: Identifying **sensitive directories** and mapping site structure via the root directory file during initial reconnaissance.,

Identify restricted paths for all user-agents.

```
User-agent: *
Disallow: /private/
```

Locate potential **admin panels** or restricted internal directories.,

```
Disallow: /admin/
```

Override broad disallow rules for specific sub-paths.

```
Allow: /public/
```

Locate the XML sitemap for efficient site structure mapping.

```
Sitemap: https://<DOMAIN>/sitemap.xml
```

Determine server sensitivity to request volume via the **crawl-delay** directive.

```
Crawl-delay: 10
```

Dangerous / misconfigured settings:

- Listing the exact path to **private content** or admin interfaces in a publicly accessible text file.
- Assuming directives provide security when **robots.txt is not strictly enforceable**.

Gotchas:

- **Directives serve as a map for attackers** even though legitimate search engine bots respect them for etiquette.