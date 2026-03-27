**Cloud Resources (Passive)**

Misconfigured cloud storage = unauthenticated access. Always check.

**Main targets:**

- AWS → S3 buckets (`s3-website-*.amazonaws.com`)
- Azure → Blobs (`blob.core.windows.net`)
- GCP → Cloud Storage

---

**Finding cloud storage**

DNS resolution often leaks it directly (from previous subdomain enumeration):

```
s3-website-us-west-2.amazonaws.com 10.129.95.250
```

**Google Dorks:**

```
intext:<company> inurl:amazonaws.com
intext:<company> inurl:blob.core.windows.net
```

**Other sources:**

- Page source code — images/JS/CSS often loaded from cloud buckets
- **domain.glass** — infrastructure overview, also reveals Cloudflare (note for Layer 2 Gateway)
- **GrayHatWarfare** — search/filter public cloud buckets by provider and file type

---

**What to look for in buckets:**

- PDFs, docs, presentations, code
- `id_rsa` / `id_rsa.pub` — leaked SSH keys = direct login, no password needed
- Config files, credentials

**Company name abbreviations** are commonly used for bucket names — try variations when searching.