# FinalRecon: Automated Web Reconnaissance

Automating web reconnaissance enhances **efficiency and accuracy**, allowing for information gathering **at scale** and more **rapid identification** of potential vulnerabilities compared to manual methods. Frameworks like **FinalRecon** provide a suite of tools to streamline this process.

### Installation Workflow

Follow these steps to set up the environment and ensure all dependencies are met:

1. **Clone the repository** from GitHub to create the local tool directory.
    
    ```
    git clone https://github.com/thewhiteh4t/FinalRecon.git
    ```
    
2. **Navigate into the directory**.
    
    ```
    cd FinalRecon
    ```
    
3. **Install Python dependencies** to ensure the tool has required libraries and modules.
    
    ```
    pip3 install -r requirements.txt
    ```
    
4. **Modify file permissions** to make the main script executable.
    
    ```
    chmod +x ./finalrecon.py
    ```
    
5. **Verify the installation** and view available modules.
    
    ```
    ./finalrecon.py --help
    ```
    

---

### Command Reference

#### Primary Recon Modules

|Option|Argument|Description|
|:--|:--|:--|
|`--url`|`<TARGET_URL>`|Specifies the target URL for analysis.|
|`--headers`|N/A|Retrieves **header information** (e.g., Server type, Power-by tags).|
|`--sslinfo`|N/A|Collects **SSL certificate** details.|
|`--whois`|N/A|Performs a **Whois lookup** for domain registration data.|
|`--crawl`|N/A|**Crawls** the target website.|
|`--dns`|N/A|Performs **DNS enumeration**.|
|`--sub`|N/A|Enumerates **subdomains**.|
|`--dir`|N/A|Searches for **directories** on the target.|
|`--wayback`|N/A|Retrieves **Wayback URLs** to view historical snapshots.|
|`--ps`|N/A|Executes a **fast port scan**.|
|`--full`|N/A|Initiates a **comprehensive reconnaissance** scan using all modules.|

#### Global Options & Performance

|Option|Argument|Default|Description|
|:--|:--|:--|:--|
|`-dt`|`<NUMBER>`|30|Threads for **directory enumeration**.|
|`-pt`|`<NUMBER>`|50|Threads for **port scanning**.|
|`-w`|`<PATH>`|`wordlists/dirb_common.txt`|Path to custom **wordlist**.|
|`-e`|`<EXTENSIONS>`|N/A|Specific **file extensions** (e.g., txt, php).|
|`-o`|`<FORMAT>`|txt|**Export format** for results.|
|`-k`|`<API_KEY>`|N/A|Adds third-party **API keys** (e.g., Shodan).|

---

### Operational Execution

#### Targeted Metadata Gathering

Use specific flags when you only need high-level infrastructure details, such as **web server versions** or **domain ownership**.

```
./finalrecon.py --headers --whois --url <TARGET_URL>
```

- **Attack Implication:** Identifying the server version (e.g., `Apache/2.4.41`) through headers allows for targeted exploit research.
- **Data Export:** Results are automatically saved to the local export directory (Default: `~/.local/share/finalrecon/dumps/`) for later review.

#### Full Reconnaissance

Use when starting a fresh engagement to build a complete profile of the target's external footprint.

```
./finalrecon.py --full --url <TARGET_URL>
```

---

### Configuration & Edge Cases

|Setting|Status|Impact|
|:--|:--|:--|
|`--s`|**Toggle SSL Verification**|Enabled by default; may need to be disabled for self-signed certificates.|
|`--r`|**Allow Redirect**|Disabled by default; if the target redirects to a different domain/path, the scan may stop unless this is toggled.|