[MODE: full]

1. Attempt SMB NULL session or LDAP anonymous bind for user list.
2. If discovery fails, scrape LinkedIn and combine with statistically-likely-usernames lists.
3. Search Google for organization-published PDFs and check Author field metadata for internal **GUID** patterns.
4. Generate potential usernames using Bash if patterns (e.g., F9L8) are identified.
5. Perform user enumeration via Kerbrute to validate the list against the domain.
6. Identify domain password policy (attempts before **Lockout**, reset threshold) before spraying.
7. Execute password spray with one common password across all validated usernames.
8. Introduce a **DELAY** between different password attempts to avoid triggering **Lockout**.
9. If successful, use access to run BloodHound or identify attack paths like RBCD or Shadow Credentials.

---

## Username Generation

Identify pattern in PDF metadata Author fields and generate a complete list of potential accounts.

```
for x in {{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}; do echo $x; done
```

- **Dangerous / misconfigured settings**
    
    - Predictable **GUID** structures in internal usernames.
    - Organization failure to scrub metadata from public-facing documents.
- **Gotchas**
    
    - **Incomplete user lists** derived from standard repos like jsmith.txt often only identify 40-60% of valid accounts.

## Password Spraying

Attempt one common password against a high volume of usernames when no other **Foothold** exists.

Use Kerbrute to identify valid accounts and attempt authentication with a single password.

```
kerbrute <COMMAND> -d <DOMAIN> --dc <DC_IP> <USER_LIST_FILE> <PASSWORD>
```

> ⚠️ Gap: The source mentions using Kerbrute for enumeration and spraying but does not provide the specific subcommands (e.g., `userenum`, `passwordspray`).

- **Edge cases**
    
    - Use a single "hail mary" attempt with one weak password if the lockout policy cannot be determined.
    - Consult the client for the password policy during an engagement if no **Foothold** allows for internal enumeration.
- **Gotchas**
    
    - **Account Lockout** will occur if multiple password attempts are made without a **DELAY** or knowledge of the lockout threshold.
    - **Unknown password policy** requires a wait of several hours between attempts to ensure the threshold resets.