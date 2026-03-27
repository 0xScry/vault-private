# Writing Custom Wordlists and Rules

### Password Policy Circumvention

**Password policies** (requiring uppercase, numbers, special characters, and minimum lengths) are often undermined by predictable human behavior. Users typically choose passwords based on the service name, company name, or personal interests (pets, hobbies, sports).

Use **OSINT** to identify these personal details to create targeted wordlists.

|Description|Password Syntax Example|
|:--|:--|
|First letter uppercase|`Password`|
|Adding numbers|`Password123`|
|Adding year|`Password2022`|
|Adding month|`Password02`|
|Ending with exclamation mark|`Password2022!`|
|Special character substitution|`P@ssw0rd2022!`|

_Note: Most passwords are 10 characters or fewer. When organizations require regular changes, users often increment a digit or change the month name._

---

### Generating Base Wordlists with CeWL

Use **CeWL** to spider a target’s website and extract unique words. This creates a list of keywords relevant to the organization's industry and culture, which are likely candidates for user passwords.

**Operational Workflow:**

1. Identify the target website.
2. Run CeWL to crawl the site for unique strings.
3. Use the resulting list as a base for mutation rules.

**Command Reference:**

|Parameter|Description|
|:--|:--|
|`-d <DEPTH>`|The depth to spider the website.|
|`-m <LENGTH>`|Minimum length of words to extract.|
|`--lowercase`|Store all discovered words in lowercase.|
|`-w <FILE>`|Path to the output file.|

```
cewl <URL> -d <DEPTH> -m <MIN_LENGTH> --lowercase -w <WORDLIST>
```

---

### Hashcat Rule-Based Mutations

Rule-based attacks allow you to transform a base wordlist into thousands of variants that satisfy complexity requirements. This is used when simple wordlists fail against systems enforcing complex password policies.

**Common Hashcat Rule Functions:**

|Function|Description|
|:--|:--|
|`:`|Do nothing (keep original word).|
|`l`|Lowercase all letters.|
|`u`|Uppercase all letters.|
|`c`|Capitalize the first letter and lowercase the rest.|
|`sXY`|Replace all instances of character `X` with character `Y`.|
|`$!`|Append an exclamation point `!` to the end of the word.|

**Generating the Mutated Wordlist:**

1. Create a rule file (e.g., `custom.rule`) with one transformation rule per line.
2. Apply the rule file to a base wordlist using the `--stdout` flag to preview or save the results.

```
hashcat --force <BASE_WORDLIST> -r <RULE_FILE> --stdout | sort -u > <MUTATED_WORDLIST>
```

**Attack Implications:**

- **Pre-built rules:** Tools like Hashcat and John the Ripper (JtR) include `best64.rule`, a highly effective set of common transformations.
- **Effectiveness:** Targeted guessing is significantly more successful when rules are combined with info regarding the **geographical region**, **industry**, and **password policy**.