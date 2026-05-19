## Proxy Extension Management

### Burp Suite BApp Store

Core functionality lacks specific vulnerability checks or protocol support.

Navigate to the BApp Store sub-tab to view available community extensions.

```
Extensions -> BApp Store -> Sort by Popularity
```

- Burp Suite → `Extensions > BApp Store` → Prefer for specialized enterprise scanners and Pro-exclusive features.
- OWASP ZAP → `Manage Add-ons > Marketplace` → Prefer for open-source community plugins and integrated wordlist packs.

> ⚠️ Gap: Python-based extensions require the Jython standalone JAR path to be manually configured in the Extensions settings or they will fail to load.

- **Pro-only extensions** will not install or function on Community Edition.

### ZAP Marketplace

Fuzzer or active scanner requires updated community modules or external wordlists.

Access the Marketplace to install stable or experimental add-ons.

```
Manage Add-ons -> Marketplace -> Check: Release/Beta/Alpha status
```

- **Beta or Alpha builds** may experience execution errors or instability during automated scans.

---

## Protocol and Data Transformation

Standard decoding tools fail to handle specific hashes or hex-manipulation requirements.

Install Decoder Improved for MD5 hashing and hex editor support.

```
Extensions -> BApp Store -> Decoder Improved -> Install
```

Input text and select hashing algorithm for quick credential or token analysis.

```
Decoder Improved -> Input: <STRING> -> Hash With -> MD5
```

---

## Automated Payload Integration

Targeted fuzzing requires specialized payloads to bypass WAFs or test for OS command injection.

Install FuzzDB modules to expand local fuzzer wordlists.

```
Marketplace -> FuzzDB Files -> FuzzDB Offensive -> Install
```

Select specific attack wordlists within the ZAP fuzzer interface.

```
Fuzz -> File Fuzzers -> fuzzdb -> attack -> os-cmd-execution
```

- **Release status** add-ons should be prioritized over Alpha/Beta versions for engagement stability.