## Methodology - Search Engine Discovery

1. Identify the primary public surface area of `<DOMAIN>` to establish a baseline of indexed content.
2. Isolate administrative or non-public interfaces by filtering out known standard pages using exclusion operators.
3. Target high-value file extensions and specific naming conventions to find leaked internal documentation or configurations.
4. Search for sensitive strings and exact phrases in page bodies and titles to identify potential credential exposures or "password reset" functionality.
5. Pivot to historical or related data if live content is restricted or has been recently modified to evade detection.

---

## Initial Domain Scoping

**When to use** — Base reconnaissance requires mapping the publicly accessible footprint of `<DOMAIN>` where standard crawling might be blocked or restricted.

List all publicly indexed pages associated with a specific target domain

```
site:<DOMAIN>
```

Exclude a specific subdomain or path from the results to find hidden or non-standard entries

```
site:<DOMAIN> -site:<SERVICE_NAME>.<DOMAIN>
```

- **Gotchas** — **Search engines do not index all information**, meaning a lack of results is not a confirmation that the resource does not exist.

## Interface Discovery

**When to use** — Mapping the directory structure or locating management portals like login pages and admin panels when no direct links exist on the homepage.

Identify pages with specific administrative terms in the URL path

```
inurl:<SERVICE_NAME>
```

Force results to contain multiple specific terms in the URL for more precise portal discovery

```
allinurl:admin panel
```

Locate specific management or classification terms in the HTML title tag

```
intitle:"admin panel"
```

- **Edge cases** — Use `allintitle:` when looking for specific dated reports or multi-word administrative headers to reduce noise from generic titles.
- **Gotchas** — **Syntax varies slightly between engines**, which can result in inconsistent behavior if shifting between Google, Bing, or DuckDuckGo.

## Sensitive File Harvesting

**When to use** — Identifying exposed internal documents, manuals, or configuration files that are indexed but not intended for public access.

Search for specific file extensions on the target domain to find downloadable assets

```
site:<DOMAIN> filetype:pdf
```

Use wildcards to find variations of user or technical manuals that may contain environment details

```
site:<DOMAIN> filetype:pdf user* manual
```

- **Tool comparison**
    - `filetype:` → `filetype:pdf` → Use for standard extension-based discovery.
    - `ext:` → `ext:pdf` → Use as a functional equivalent depending on engine support.

## Credential and String Hunting

**When to use** — Searching for leaked credentials, "password reset" pages, or specific "confidential" markings within the body text of indexed pages.

Locate specific sensitive strings within the body text of a page

```
intext:"password reset"
```

Search for multiple specific terms in the body text to narrow down leak locations

```
allintext:admin password reset
```

Target exact phrases to find specific policy documents or internal headers

```
"information security policy"
```

- **Gotchas** — **Data may be deliberately hidden or protected**, causing operators to return no results even if the sensitive strings exist on the server.

## Historical and External Pivot

**When to use** — Accessing content that has been removed from the live site or identifying third-party relationships and similar organizations.

View the indexed version of a page to see content that may have been recently updated or deleted

```
cache:<DOMAIN>
```

Identify external websites that are linking back to the target domain to find business partners

```
link:<DOMAIN>
```

Discover websites with similar content or functionality to the target

```
related:<DOMAIN>
```

> ⚠️ Gap: **Search engine discovery relies entirely on third-party indexing**, so any technique targeting the live environment will fail if the site uses `robots.txt` or `noarchive` tags to prevent indexing.

## Advanced Filtering and Logic

**When to use** — Refining broad result sets to exclude noise or targeting specific numerical data like version numbers or price ranges.

Narrow results by requiring all specified terms to be present across different fields

```
site:<DOMAIN> AND (inurl:admin OR inurl:login)
```

Identify pages containing numerical ranges such as versions, dates, or prices

```
site:<DOMAIN> "price" 100..500
```

Alternative range search for finding specific numerical identifiers

```
site:<DOMAIN> numrange:1000-2000
```

- **Edge cases** — Use the `-` sign instead of `NOT` for more compact exclusion queries when searching for non-standard content paths.