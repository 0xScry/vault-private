- Identify if authentication relies solely on **passwords** or incorporates factors like CAC, PIN, or biometrics.
- Prioritize **password** guessing using common patterns: `123456`, `qwerty`, `password`, or personal names.
- Upon compromising one service, immediately attempt authentication on all other identified platforms using the same `<PASSWORD>`.
- Query known `<EMAIL_ADDRESS>` targets against breach databases to find leaked credentials that likely remain valid due to poor password hygiene.

---

## Weak Password Guessing

Use when MFA is absent and users are permitted to set simple credentials. 24% of users rely on a small set of predictable strings.

Common weak password candidates for wordlists

```
123456
qwerty
password
<USERNAME>
<PET_NAME>
<CHILD_NAME>
```

**Dangerous / misconfigured settings**

- Lack of complexity requirements allowing 8-digit numeric or alpha-only strings.
- No account lockout or rate-limiting on **authentication** endpoints.

**Gotchas** **123456** remains the most prevalent credential in data breaches with 4.5 million occurrences.

## Credential Reuse

Use after recovering a valid `<PASSWORD>` for a `<USERNAME>` or `<EMAIL_ADDRESS>`. There is a 66% statistical probability that the credential functions across multiple platforms.

Manual reuse attempt on secondary service

```
<SERVICE_NAME> login: <USERNAME> / <PASSWORD>
```

> ⚠️ Gap: Source establishes the high probability of reuse but lacks a specific tool or protocol for automated spraying.

**Edge cases**

- Users employing **password managers** (36% adoption) are significantly less susceptible to reuse attacks.

**Gotchas** **55% of users** fail to change passwords following a confirmed data breach.

## Breach Identification

Use to find historical leaks associated with a specific `<EMAIL_ADDRESS>` to identify potential valid credentials.

Check for presence in reported data breaches

```
https://haveibeenpwned.com/
```

**Gotchas** **Successful identification** in a breach does not guarantee the current password matches the leaked one, though the 55% non-change rate makes it a high-value lead.