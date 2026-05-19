## METHODOLOGY

1. Run standard **Spider** to map accessible endpoints and directories
2. Run **Ajax Spider** if the application utilizes JavaScript for dynamic content loading to capture missed links
3. Analyze **Passive Scanner** alerts for surface-level vulnerabilities like missing security headers or **DOM-based XSS** during the crawl
4. Initiate **Active Scan** to probe parameters for server-side flaws once the **Sites Tree** is populated
5. Verify **High** alerts manually using the **Request Editor** or **ZAP HUD** to confirm findings like **Remote OS Command Injection**
6. Export results via **Generate HTML Report** for documentation

---

## Automated Site Mapping

When to use: Initial discovery phase to identify sub-directories and file structures before active testing

Start crawl from specific request in history

```
Attack > Spider
```

Start crawl via HUD browser interface

```
Spider Start
```

Tool comparison

- **ZAP Spider**
    - `Attack > Spider`
    - Prefer for standard HTML link discovery and rapid mapping
- **Ajax Spider**
    - `Third button on right pane`
    - Prefer for identifying links requested through **JavaScript AJAX** requests after page load

Dangerous / misconfigured settings

- **Target out of scope**: ZAP will ignore URLs unless explicitly added to **Scope** during the prompt or via settings

Gotchas

- **HUD browser incompatibility**: **ZAP HUD** may fail to function in specific browser versions
- **Incomplete Ajax crawl**: Always run normal **Spider** before **Ajax Spider** for comprehensive coverage

## Passive Vulnerability Identification

When to use: Real-time monitoring of traffic to identify configuration flaws without sending additional attack payloads

Review identified issues in the UI

```
Alerts tab
```

Edge cases

- Multiple page alerts: The left pane displays alerts for the **current page** only, while the right pane shows **overall alerts** for the entire application

Gotchas

- **False sense of security**: Passive alerts only identify issues visible in **source code** or **HTTP headers**; they do not confirm exploitable server-side logic

## Active Vulnerability Scanning

When to use: Automated probing of HTTP parameters and endpoints for exploitable vulnerabilities like **Remote OS Command Injection**

Execute attack against identified site tree

```
Active Scan
```

Manual verification of injection vulnerability

```
<TARGET_IP>&cat /etc/passwd&
```

Gotchas

- **Incomplete site tree**: **Active Scan** targets only identified pages; failing to run **Spider** first results in skipped endpoints
- **Time consumption**: **Active Scanner** takes significantly longer than passive or spidering phases due to the volume of attack payloads

## Reporting Findings

When to use: Compiling scan data into a formatted log for documentation or client delivery

Generate web-viewable summary

```
Report > Generate HTML Report
```

Edge cases

- Format requirements: Export in **XML** or **Markdown** if data needs to be parsed by other tools or integrated into different note systems

> ⚠️ Gap: The source does not specify how to handle authentication (session cookies, headers) during spidering or active scanning, which will cause the scanner to fail silently on any endpoints behind a login wall.