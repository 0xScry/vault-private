## Web Crawling

When to use: Automated link discovery is required to map site structure and index pages starting from a seed URL.

Commands Set the initial target for the crawler logic

```
http://<TARGET_IP>/
```

Dangerous / misconfigured settings

- **Directory browsing enabled** on sensitive paths like `/files/`
- **Publicly accessible** file servers referenced in source code

Gotchas

- **Fuzzing** is distinct from crawling; crawlers follow existing links rather than guessing potential ones.

> ⚠️ Gap: Source lacks specific tool execution (e.g., Burp Suite Spider, Scrapy, or CLI crawlers) and required flags for depth/concurrency limits.

---

## Breadth-First Strategy

When to use: Site mapping requires a broad overview of all top-level content before investigating deep sub-directories.

Dangerous / misconfigured settings

- **Exposed backup archives** or internal documents in directory listings discovered during broad sweeps

Gotchas

- **Shallow crawling** may miss specific, deeply nested content or isolated directories.

---

## Depth-First Strategy

When to use: Target investigation requires reaching deep into the site's internal structure or following specific single paths to their conclusion.

Edge cases

- Use when looking for specific content hidden deep within nested directories.

Gotchas

- **Backtracking** is required to explore alternative paths, which can increase total crawl time on complex structures.

---

## Contextual Reconnaissance Analysis

When to use: Isolated findings like version numbers in comments or metadata require correlation to confirm exploitability.

Commands Identify patterns in extracted URLs to target manual inspection

```
/files/
```

Dangerous / misconfigured settings

- **Outdated software versions** listed in metadata or comments
- **Vulnerable configuration files** accessible via crawled links
- **Sensitive data exposure** in publicly reachable directories

Gotchas

- **Data isolation** prevents identifying critical indicators; a "file server" comment only becomes critical when correlated with an accessible `/files/` directory.