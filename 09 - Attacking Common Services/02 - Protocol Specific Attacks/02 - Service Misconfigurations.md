## Service Identification and Default Research

When to use: Service is active and reachable; target is an older application or unhardened installation.

> ⚠️ Gap: Source outlines banner grabbing and version identification as the primary workflow but provides no specific command-line tools for implementation.

Dangerous / misconfigured settings

- **Unnecessary defaults** — Settings, features, and files configured for usability rather than security.
- **Security defaults** — Accepting initial configuration values on devices or software.

Gotchas **Credential discovery** — Attackers identify weak settings and specific equipment credentials via internet searches.

## Authentication Testing

When to use: Service banner is retrieved; checking for factory settings or administrative negligence.

Common weak credential combinations for manual testing or wordlist creation

```
admin:admin
admin:password
admin:<blank>
root:12345678
administrator:Password
```

Dangerous / misconfigured settings

- **Default credentials** — Vendor-set usernames and passwords left unchanged post-installation.
- **Weak passwords** — Lack of minimum password complexity policies leading to predictable combinations.
- **Blank passwords** — Administrators omitting passwords during initial setup and failing to update them.

Gotchas **Legacy persistence** — Older applications are significantly more likely to contain unchanged default credentials than modern software.

## Anonymous Authentication

When to use: Network connectivity is confirmed; testing for access without any credential prompt.

Dangerous / misconfigured settings

- **Unauthenticated access** — Service configuration allows any user with connectivity to enter without a password.

## Access Rights and Information Harvesting

When to use: Valid credentials obtained; checking for horizontal or vertical privilege expansion.

Dangerous / misconfigured settings

- **Misconfigured access rights** — User accounts assigned incorrect permissions or over-permissive roles.
- **Broken RBAC/ACL** — Lower-level personnel granted access to private managerial or administrative information.

Gotchas **Information chaining** — Access to a single over-privileged service like FTP can reveal configuration files and plain text credentials for other infrastructure components.