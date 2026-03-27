# Hashcat Methodology and Operational Guide

**Hashcat** is an open-source password recovery tool supporting Linux, Windows, and macOS that utilizes **GPU support** to crack a wide variety of hashes through various attack modes.

### Hash Identification and Preparation

Before initiating an attack, the specific **hash type** and its corresponding **ID** must be identified.

1. **Manual Identification:** Reference the Hashcat website's example hashes to compare the target hash format with known types.
2. **Automated Identification:** Use `hashid` to determine the specific **Hashcat Mode** ID.
    
    ```
    hashid -m '<HASH>'
    ```
    
3. **Help Menu:** To view a complete list of supported hash types and their associated IDs, use the help flag.
    
    ```
    hashcat --help
    ```
    

|Common Hash Types|Mode ID|
|:--|:--|
|MD5|0|
|MD4|900|
|SHA1|100|
|SHA2-256|1400|
|SHA2-512|1700|
|MD5 Crypt|500|

---

### Dictionary Attack (`-a 0`)

**Use Case:** Use this mode when you have a target hash and a wordlist (e.g., `rockyou.txt`) containing potential passwords.

**Workflow:**

1. Identify the hash type and ID.
2. Execute the attack by providing the hash and the path to the wordlist.

```
hashcat -a 0 -m <HASH_ID> <HASH> <WORDLIST>
```

---

### Rule-Based Attack

**Use Case:** Apply this when a standard wordlist attack fails. **Rules** perform specific modifications to each word in a list—such as appending numbers or leet-speak substitutions—to expand the search space without requiring a larger wordlist.

**Workflow:**

1. Identify a relevant ruleset (typically found in `/usr/share/hashcat/rules`).
2. Append the `-r` flag to the dictionary attack command to apply the transformations.

```
hashcat -a 0 -m <HASH_ID> <HASH> <WORDLIST> -r /usr/share/hashcat/rules/<RULESET>.rule
```

**Common Ruleset:**

- `best64.rule`: Contains 64 standard modifications, including character substitution and appending digits.

---

### Mask Attack (`-a 3`)

**Use Case:** Use for **targeted brute-force** when the password structure (keyspace) is partially known. For example, if a password is known to be eight characters long with a specific pattern of letters and numbers, a mask limits the attempt to only those combinations.

**Workflow:**

1. Define the password pattern using built-in or custom character sets.
2. Execute the attack using the `-a 3` mode and the defined mask.

```
hashcat -a 3 -m <HASH_ID> <HASH> '<MASK>'
```

**Built-in Character Sets:**

|Symbol|Charset|
|:--|:--|
|`?l`|abcdefghijklmnopqrstuvwxyz|
|`?u`|ABCDEFGHIJKLMNOPQRSTUVWXYZ|
|`?d`|0123456789|
|`?h`|0123456789abcdef|
|`?s`|Special characters/symbols|
|`?a`|Combination of `?l?u?d?s`|

**Example Scenario:** To target a password starting with one uppercase letter, followed by four lowercase letters, one digit, and one symbol (e.g., `P@ss1!`), the mask would be: `?u?l?l?l?l?d?s`.

**Custom Character Sets:** Define custom sets using `-1`, `-2`, `-3`, or `-4` and reference them in the mask as `?1`, `?2`, etc.