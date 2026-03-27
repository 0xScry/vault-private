# Password Protection and Management

### Password Manager Fundamentals

**Password managers** are applications that store credentials and sensitive information in an **encrypted database**, typically secured by a **master password**. They provide a centralized way to maintain strong, unique credentials across multiple services.

**Operational Workflow: Database Access**

1. User enters **master password**.
2. System uses **cryptographic hash functions** and **key derivation functions (KDF)** to derive encryption keys.
3. The database is decrypted, granting access to stored credentials.

---

### Cloud-Based Password Managers

Use when **synchronization** across multiple devices (e.g., Linux, Android, Chrome OS) is required for different websites and applications.

#### Implementation: Zero-Knowledge Encryption

This approach ensures the service provider cannot access user data.

- **Key Derivation:** Encryption keys are derived directly from the master password.
- **Algorithms:** Often utilizes **PBKDF2-SHA256** for master key derivation and **AES-256 bit** encryption for vault access.

**Common Cloud Providers:**

- Bitwarden
- 1Password
- LastPass
- Dashlane

---

### Local Password Managers

Use when an organization or individual prefers to avoid third-party services and maintains full responsibility for the **storage location** and **database protection**.

#### Security Mechanisms

- **Data Storage:** The encrypted database is stored strictly on the **local system**.
- **Protection against Precomputed Keys:** Employs KDFs with a **random salt** to hinder dictionary and guessing attacks.
- **Advanced Local Protections:**
    - **Memory Protection:** Prevents unauthorized access to data in RAM.
    - **Keylogger Resistance:** Utilizes secure desktop environments (similar to Windows UAC) to protect input.

**Common Local Providers:**

- KeePassXC
- KeePass
- Strongbox

---

### Comparison of Password Management Approaches

|Feature|Cloud-Based|Local-Based|
|:--|:--|:--|
|**Data Storage**|Provider's Cloud|Local System|
|**Sync Capability**|Native Multi-device Sync|User-managed|
|**Encryption Responsibility**|Shared (Vendor implementation)|User-led|
|**Access Model**|Zero-Knowledge Encryption|Local Decryption|
|**Key Protections**|PBKDF2, AES-256|Hashing, Salting, Memory Protection|

---

### Passwordless Authentication

Use to eliminate the risks associated with **knowledge factors** (passwords), such as theft, sharing, and repeat use.

#### Authentication Factors

- **Possession Factor:** Something the user **has**.
- **Inherent Factor:** Something the user **is** (biometrics).

#### Common Identity Providers

- Microsoft
- Auth0
- Okta
- Ping Identity

---

### Attack Surface and Implications

|Vulnerability|Context/Implication|
|:--|:--|
|**Knowledge Factor Reliance**|Vulnerable to **cracking**, **guessing**, and **shoulder surfing**.|
|**Password Reuse**|Simplifies unauthorized access across multiple services if one is compromised.|
|**Precomputed Keys**|Mitigated by using **random salts** in local managers.|
|**Keylogging**|Can capture master passwords; mitigated by **secure desktop environments**.|

_Note: No specific technical commands were provided in the source text for copy-pasting._