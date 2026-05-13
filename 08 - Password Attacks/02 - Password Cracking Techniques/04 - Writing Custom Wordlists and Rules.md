
1. Conduct OSINT on the organization or individual to identify company names, geographical regions, pets, hobbies, and interests.
2. Scrape the target's web presence using CeWL to build a base wordlist of industry-specific terms.
3. Identify the target's password policy to determine minimum length and required character sets (e.g., uppercase, numbers, special characters).
4. Create or select a Hashcat ruleset (e.g., `best64.rule` or a custom `.rule` file) that mirrors common human behavior, such as appending the current year or month.
5. Apply the ruleset to the base wordlist to generate mutated variants that satisfy the identified policy.
6. Execute the cracking attempt against obtained hashes using the mutated wordlist.

---

## Targeted Wordlist Generation

Web presence exists for a corporate target and you need words with a higher probability of being used by employees.

Scrape words from a website with specific depth and minimum length constraints

```
cewl <URL> -d <DEPTH> -m <MIN_LENGTH> --lowercase -w <FILE_PATH>
```

- **Gotchas**: **Small wordlists** often result from setting the minimum length (`-m`) too high or depth (`-d`) too low for the specific site structure.

## Rule-Based Mutation

Target password policy requires complexity (uppercase, digits, special characters) and users are likely following predictable patterns like appending years or capitalizing the first letter.

Apply custom rules to a wordlist and output unique mutated strings to a new file

```
hashcat --force <FILE_PATH> -r <FILE_PATH> --stdout | sort -u > <FILE_PATH>
```

### Hashcat Rule Syntax

Functions to define transformations within a `.rule` file:

- `:` Do nothing
- `l` Lowercase all letters
- `u` Uppercase all letters
- `c` Capitalize first letter, lowercase others
- `sXY` Replace all instances of X with Y
- `$!` Append "!" to the end of the word

### Tool comparison

- Custom Rules -> `custom.rule` -> prefer when you have specific OSINT (e.g., target always uses specific years or suffixes).
    
- Pre-built Rules -> `best64.rule` -> prefer for general-purpose cracking when specific user patterns are unknown.
    
- **Gotchas**: **Ruleset mismatch** occurs when the transformations do not account for the specific password policy, such as failing to add a required special character.
    

## Password Pattern Analysis

Target requires regular password changes, causing users to increment digits or change months/years in their existing passwords.

Common syntax patterns to include in custom rules:

```
c             # First letter uppercase
$1$2$3        # Adding numbers
$2$0$2$2      # Adding year
$!            # Adding special character
```

- **Edge cases**: Most passwords are no longer than ten characters; focus rules on base words of at least five characters to meet typical 8-10 character requirements.
- **Gotchas**: **Guessing game failure** is the baseline; cracking will fail if keywords related to industry, geography, or personal OSINT are overlooked.