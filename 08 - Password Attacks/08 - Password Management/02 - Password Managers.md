## Cloud-Based Vault Decryption

When to use: Assessing Zero-Knowledge systems where the service provider has no access to the plaintext database.

Master key derivation and decryption flow for Bitwarden-style implementations

```
PBKDF2-SHA256 -> Master Key -> AES-256
```

> ⚠️ Gap: Source lacks the specific hashing parameters (iterations) and salt derivation methods required to replicate the master key for offline cracking.

**Gotchas** **Zero-Knowledge Encryption** shifts the attack surface entirely to the master password strength and the local implementation of the KDF.

---

## Local Vault Security

When to use: Target database is stored on the local filesystem and bypasses third-party cloud synchronization to reduce data transmission risks.

**Dangerous / misconfigured settings**

- **Missing random salt** in key derivation functions allows for the use of precomputed keys and facilitates dictionary attacks.
- **Disabled memory protection** or lack of a secure desktop environment/UAC increases vulnerability to local keyloggers.

**Gotchas** **Local storage responsibility** is placed entirely on the user, meaning compromise of the local system or storage location leads to direct database access.

---

## Passwordless Authentication

When to use: Identifying alternatives to knowledge-based factors to mitigate cracking, guessing, and shoulder surfing.

**Edge cases**

- **Inherent factors** (biometrics) or **possession factors** (hardware tokens) are required when a knowledge factor (password) is completely removed from the authentication flow.

**Gotchas** **Knowledge factor reliance** remains the primary point of failure for legacy systems, necessitating third-party identity providers to enforce passwordless standards.