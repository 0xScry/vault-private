### Web Reconnaissance: .well-known URIs

The **.well-known** standard (RFC 8615) defines a standardized directory (`/.well-known/`) located in a website's root domain. It centralizes **critical metadata**, configuration files, and information regarding services, protocols, and security mechanisms.

#### Operational Workflow: Metadata Discovery

1. **Identify the target domain.**
2. **Request specific standardized URIs** to retrieve configuration details or security policies.
3. **Analyze the response** (often JSON or text) to map the attack surface, identify authentication providers, or find administrative endpoints.

#### Common .well-known URI Reference

Use these endpoints to discover security contacts, authentication configurations, and domain ownership details.

|URI Suffix|Description|Recon/Attack Value|
|:--|:--|:--|
|`security.txt`|Contact info for reporting vulnerabilities.|Identify security personnel or disclosure policies.|
|`change-password`|Standard URL for password change pages.|Directly locate account management functionality.|
|`openid-configuration`|Details for OpenID Connect (OIDC).|Map OAuth 2.0 endpoints and authentication methods.|
|`assetlinks.json`|Verification for digital assets/apps.|Confirm domain ownership and associated mobile applications.|
|`mta-sts.txt`|SMTP MTA Strict Transport Security policy.|Identify email security and transport policies.|

---

### Technique: OpenID Connect Discovery

**Scenario:** Use when a target application utilizes OpenID Connect (built on OAuth 2.0) for identity management. This is a primary method for mapping the **identity landscape** of a target.

**Command Reference:**

```
curl https://<DOMAIN>/.well-known/openid-configuration
```

**Attack Implications:** Retrieving this JSON document unlocks several exploration opportunities by revealing metadata about the provider:

- **Endpoint Mapping:** Identifies `authorization_endpoint`, `token_endpoint`, and `userinfo_endpoint`.
- **Cryptographic Material:** Locates the `jwks_uri` containing public keys used for token signing.
- **Supported Methods:** Lists `response_types_supported` (e.g., code, token) and `scopes_supported` (e.g., openid, profile, email), which informs subsequent **OAuth/OIDC exploitation** attempts.

---

### Methodology Notes

- **Discovery Automation:** Clients and security tools use these paths to automatically locate configuration files without manual searching.
- **Standardization:** Using the **IANA registry** for .well-known URIs ensures a consistent approach to uncovering structured metadata across different web environments.
- **Decision Point:** If a website uses third-party authentication, checking `openid-configuration` should be a priority to understand the trust relationship and token handling.