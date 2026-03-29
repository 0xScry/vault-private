# Web Reconnaissance Fundamentals

Web Reconnaissance is the foundational **Information Gathering** phase of a penetration test, occurring after pre-engagement and before vulnerability assessment. It involves the systematic collection of data to tailor attacks against specific weaknesses.

## 1. Active Reconnaissance

**When to use:** Use when a comprehensive view of the target's infrastructure is required and the risk of detection is acceptable. **Why it matters:** Direct interaction provides the most accurate data on service versions and configurations, unlocking specific exploitation paths.

### Active Techniques and Methodology

|Technique|Goal|Tools|Risk of Detection|
|:--|:--|:--|:--|
|**Port Scanning**|Identify open ports/services to find potential entry points.|`nmap`, `masscan`, `unicornscan`|**High**: Triggers IDS/Firewalls.|
|**Vulnerability Scanning**|Probe for known flaws (SQLi, XSS, outdated software).|`nessus`, `openvas`, `nikto`|**High**: Sends detectable exploit payloads.|
|**Network Mapping**|Map topology and infrastructure hops.|`traceroute`, `nmap`|**Med-High**: Unusual traffic patterns.|
|**Banner Grabbing**|Identify software versions via service headers.|`netcat`, `curl`|**Low**: Minimal interaction, but logged.|
|**OS Fingerprinting**|Determine the target operating system (Windows, Linux, etc.).|`nmap`, `xprobe2`|**Low**: Advanced methods may be detected.|
|**Service Enumeration**|Determine exact versions of running services.|`nmap`|**Low**: Can be logged.|
|**Web Spidering**|Map site structure and discover hidden resources.|`burpsuite`, `zaproxy`, `scrapy`|**Low-Med**: Detectable if traffic is not human-like.|

### Operational Commands (Active)

|Action|Command|Purpose|
|:--|:--|:--|
|**OS Detection**|`nmap -O <TARGET_IP>`|Identify the underlying OS.|
|**Service Versioning**|`nmap -sV <TARGET_IP>`|Identify specific service versions (e.g., Apache 2.4.50).|
|**Banner Grabbing**|`curl -I <TARGET_IP>`|Examine HTTP headers for software identification.|

---

## 2. Passive Reconnaissance

**When to use:** Use as the first step in information gathering to maintain stealth and avoid triggering security alerts. **Why it matters:** Leverages publicly available data to identify subdomains, infrastructure, and potential social engineering targets without ever touching the target's systems.

### Passive Techniques and Methodology

|Technique|Goal|Source/Tools|Risk of Detection|
|:--|:--|:--|:--|
|**Search Engine Queries**|Find employee info, social media, or niche data.|Google, DuckDuckGo, Bing, Shodan|**Very Low**.|
|**WHOIS Lookups**|Retrieve registration details and name servers.|`whois`|**Very Low**.|
|**DNS Analysis**|Identify subdomains and mail server infrastructure.|`dig`, `nslookup`, `dnsenum`, `fierce`|**Very Low**.|
|**Web Archive Analysis**|Find historical vulnerabilities or hidden info in old site versions.|Wayback Machine|**Very Low**.|
|**Social Media Analysis**|Identify roles and potential social engineering targets.|LinkedIn, Twitter, Facebook|**Very Low**.|
|**Code Repositories**|Find exposed credentials or vulnerabilities in source code.|GitHub, GitLab|**Very Low**.|

### Operational Commands (Passive)

|Action|Command|Purpose|
|:--|:--|:--|
|**Domain Registration**|`whois <DOMAIN>`|Find registrant contact info and name servers.|
|**Subdomain Discovery**|`dig <DOMAIN>`|Enumerate subdomains and infrastructure records.|

---

## Summary of Approaches

- **Active Reconnaissance** provides high-detail, comprehensive infrastructure views but carries a **High** risk of detection due to direct interaction.
- **Passive Reconnaissance** is **Stealthy** and relies on existing public records, but may yield less comprehensive information than direct probing.