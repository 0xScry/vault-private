## Well-Known Metadata Discovery

Initial recon phase when mapping the attack surface of a root domain to locate centralized metadata, configuration files, and security policies.

Request security researcher contact information and vulnerability disclosure policies

```
curl -s https://<DOMAIN>/.well-known/security.txt
```

Locate the standardized password change interface for credential-based attacks

```
curl -s https://<DOMAIN>/.well-known/change-password
```

Identify digital asset ownership and associated mobile applications

```
curl -s https://<DOMAIN>/.well-known/assetlinks.json
```

Retrieve SMTP MTA Strict Transport Security policies to evaluate email security posture

```
curl -s https://<DOMAIN>/.well-known/mta-sts.txt
```

> ⚠️ Gap: Source lacks specific automation tools or wordlists for directory brute-forcing; manual verification against the **IANA registry** is required to identify all registered suffixes applicable to the target.

**Registry omission** results in incomplete discovery of service-specific metadata if only common suffixes are tested.

## OpenID Connect Configuration Analysis

Target identifies as an OpenID Connect Provider or utilizes OAuth 2.0 for identity management.

Retrieve the OpenID Connect Provider discovery document to map authentication endpoints and supported methods

```
curl -s https://<DOMAIN>/.well-known/openid-configuration
```

**Dangerous / misconfigured settings**

- Exposure of internal-only `authorization_endpoint` or `token_endpoint` URIs in the JSON response.
- Inclusion of weak or deprecated `id_token_signing_alg_values_supported` such as `none`.
- Over-permissive `scopes_supported` revealed to unauthenticated users.

**Endpoint leakage** occurs when discovery documents reveal administrative or development endpoints that bypass intended security perimeters.