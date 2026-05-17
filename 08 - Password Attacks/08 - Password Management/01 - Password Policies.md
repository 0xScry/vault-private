

If registration or password reset functions are accessible:

1. Attempt to set a known weak password (e.g., `password123`) to trigger **policy disclosure** error messages.
2. Document minimum length, character types, and **blacklisted words** leaked by the application error.
3. Identify if the policy allows **company-specific keywords** or predictable mutations.
4. If password expiration is confirmed, target accounts for **trailing digit increments** (e.g., `01` to `02`).

---

## Identifying Policy Complexity

Triggering error messages during password setup to map **complexity requirements** and **enforcement** mechanisms.

Verify password strength against crack time estimations:

```
PasswordMonster
```

Generate compliant but secure strings:

```
1Password Password Generator
```

**Dangerous / misconfigured settings**

- Mandating **password expiration** which forces users into predictable pattern shifts.
- Relying on **industry standards** as a sole security measure rather than a baseline.
- Failure to implement **blacklists** for company names, seasons, or common words.

**Edge cases**

- Users leveraging **passphrases** or lyrics may bypass entropy checks but remain vulnerable to **OSINT-based** wordlist attacks.

**Gotchas**

- **Policy compliance** does not equal security; users will often choose the path of least resistance like `<COMPANY_NAME>01!` to satisfy complexity.

> ⚠️ Gap: Source lacks **CLI enumeration syntax** for querying GPO password policies via RPC or LDAP.

## Active Directory Enforcement

Targeting environment-wide policy application when Active Directory is the identity provider.

Apply or inspect policy via Group Policy Objects:

```
Active Directory Password Policy GPO
```

**Dangerous / misconfigured settings**

- Lack of technical **enforcement** tools or failure to apply GPOs across all application tiers.

**Gotchas**

- **Discrepancies in application** occur if procedures do not guarantee the policy is applied to every authentication endpoint in the organization.