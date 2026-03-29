# Web Technology Fingerprinting

**Fingerprinting** identifies the technical signatures of web servers, operating systems, and software components. This allows for **tailored attacks** by exploiting vulnerabilities specific to the identified technology stack.

## Automated Fingerprinting Tools

Use these tools to combine various techniques to identify CMSs, frameworks, and infrastructure components.

|Tool|Description|Key Features|
|:--|:--|:--|
|**Wappalyzer**|Browser extension/online service|Identifies CMSs, frameworks, and analytics tools.|
|**BuiltWith**|Web technology profiler|Provides detailed technology stack reports.|
|**WhatWeb**|Command-line tool|Uses a vast database of signatures.|
|**Nmap**|Network scanner|Uses NSE scripts for specialized fingerprinting.|
|**Netcraft**|Web security service|Reports on hosting providers and security posture.|
|**wafw00f**|WAF identification tool|Detects the presence, type, and configuration of WAFs.|

---

## Methodology: Manual Banner Grabbing

**Goal:** Extract software versions and underlying technology directly from HTTP headers.

1. **Fetch Initial Headers:** Use `curl` to fetch only the head of the response.
    
    ```
    curl -I <DOMAIN>
    ```
    
2. **Follow Redirects:** If the response is a `301 Moved Permanently`, repeat the command for the new location specified in the `Location` header to find the final technology stack.
3. **Analyze Headers:** Look for `Server` (software/OS) and `X-Redirect-By` (CMS indicators like WordPress).

**Attack Implications:** Identifying specific versions (e.g., `Apache/2.4.41 (Ubuntu)`) allows you to search for version-specific exploits.

---

## Methodology: WAF Detection

**Goal:** Determine if a Web Application Firewall (WAF) is present to avoid request blocking and prepare for evasion.

1. **Install Tool:**
    
    ```
    pip3 install git+https://github.com/EnableSecurity/wafw00f
    ```
    
2. **Scan Target:**
    
    ```
    wafw00f <DOMAIN>
    ```
    

**Attack Implications:** If a WAF (e.g., Wordfence) is detected, reconnaissance attempts may be filtered. You must **adapt techniques** to bypass or evade detection mechanisms.

---

## Methodology: Software & Vulnerability Identification

**Goal:** Use automated scanners to find outdated software, insecure configurations, and common file paths.

### Nikto Operational Workflow

1. **Installation (if required):**
    
    ```
    sudo apt update && sudo apt install -y perl
    git clone https://github.com/sullo/nikto
    cd nikto/program
    chmod +x ./nikto.pl
    ```
    
2. **Execute Fingerprinting Scan:** Use `-Tuning b` to restrict the scan to Software Identification modules only, reducing unnecessary noise.
    
    ```
    nikto -h <DOMAIN> -Tuning b
    ```
    

### Analysis of Common Misconfigurations

Scanners like Nikto identify specific security weaknesses that unlock further attack vectors.

|Finding|Impact/Implication|
|:--|:--|
|**Outdated Software**|Apache versions (e.g., 2.4.41) may have known CVEs.|
|**Missing Security Headers**|Missing `HSTS` or `X-Content-Type-Options` reduces client-side protection.|
|**Common Paths Found**|Identification of `/wp-login.php` or `/license.txt` confirms CMS type (WordPress).|
|**Insecure Cookies**|Cookies created without `httponly` flags can be accessed via scripts.|
|**BREACH Attack**|`Content-Encoding: deflate` may indicate vulnerability to BREACH.|