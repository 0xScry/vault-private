## DNS-Based Discovery

Identify cloud-hosted assets by resolving a subdomain list and filtering for known cloud provider strings to find storage used for administrative purposes.

Look up IP addresses for a list of subdomains to identify cloud-hosted infrastructure

```
for i in $(cat <FILE_PATH>);do host $i | grep "has address" | grep <DOMAIN> | cut -d" " -f1,4;done
```

- **Dangerous / misconfigured settings**
    
    - Adding cloud storage endpoints directly to public DNS records for ease of management.
- **Gotchas**
    
    - Cloud storage IPs may belong to specific regions like **s3-website-us-west-2.amazonaws.com**.

## Search Engine Dorking

Target publicly indexed storage containers when searching for sensitive file types like PDFs, text documents, or source code,.

Search for company files hosted on AWS S3

```
intext: <DOMAIN> inurl:amazonaws.com
```

Search for company files hosted on Azure Blob Storage

```
intext: <DOMAIN> inurl:blob.core.windows.net
```

- **Dangerous / misconfigured settings**
    - **S3 buckets**, **blobs**, and **cloud storage** configured to be **accessible without authentication**.

## Source Code Analysis

Identify offloaded assets by inspecting HTML for links to cloud providers used to relieve web server load.

Look for DNS prefetch/preconnect links or asset references in the source

```
<link rel="dns-prefetch" href="//<DOMAIN>.blob.core.windows.net">
```

- **Edge cases**
    - Developers often host images, JavaScript, or CSS on cloud storage to optimize performance.

## Third-Party Reconnaissance

### domain.glass

Analyze infrastructure and security measures through third-party assessment tools.

1. Navigate to domain.glass and enter `<DOMAIN>`.
2. Check the Cloudflare security assessment status.
3. Review SSL certificate details and DNS names for additional infrastructure hits.

- **Gotchas**
    - A **Safe** classification for Cloudflare indicates a **security measure** exists at the gateway layer.

### GrayHatWarfare

Passively discover and filter files stored on AWS, Azure, and GCP storage.

1. Search the GrayHatWarfare database using company names or common abbreviations.
2. Apply filters to sort by specific file formats.
3. Look for sensitive filenames like **id_rsa** or **id_rsa.pub**,.

- **Tool comparison**
    
    - GrayHatWarfare → `search + filters` → prefer for systematic file discovery across multiple cloud providers.
    - Google Dorks → `inurl: + intext:` → prefer for finding specifically indexed document types.
- **Dangerous / misconfigured settings**
    
    - Publicly accessible **SSH private keys** allow passwordless login to company machines.
    - Using predictable company abbreviations within the IT infrastructure naming convention.
- **Gotchas**
    
    - Staff under high pressure frequently make **fatal errors** leading to sensitive credential leaks.