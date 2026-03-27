# Introduction to Password Cracking

Hashing is a mathematical function that transforms input bytes into a fixed-size output (e.g., **MD5**, **SHA-256**). It is designed to be **one-way**, meaning the original password should not be retrievable from the hash alone. **Password cracking** is the process of attempting to determine the original input based on the hash.

## Hash Generation Command Reference

Use these commands to generate hashes for verification or testing purposes.

|Tool|Description|Command|
|:--|:--|:--|
|`md5sum`|Generates an MD5 hash of the input|`echo -n \|
|`sha256sum`|Generates a SHA-256 hash of the input|`echo -n \|

## Password Cracking Methodology

|Technique|When to Use|Why Use It|Implications|
|:--|:--|:--|:--|
|**Rainbow Tables**|When targeting unsalted hashes|Extremely fast; uses pre-compiled maps of inputs and outputs.|Ineffective against salted hashes.|
|**Dictionary Attack**|Primary method for penetration tests|Most efficient under time constraints; uses statistically likely passwords.|Limited by the quality and size of the wordlist (e.g., `rockyou.txt`).|
|**Brute-Force Attack**|As a last resort for short passwords (<9 characters)|100% effective given enough time; attempts every possible combination.|Extremely slow for long/strong passwords; hardware dependent.|

### Defeating Rainbow Tables with Salting

**Salting** is the addition of a random sequence of bytes to a password before hashing.

- **Decision Point:** If a hash is salted, rainbow tables become impractical because the attacker must re-map the table for every possible salt value.
- **Operational Note:** Salts are not secret; they are typically **prepended** to the hash so the system can verify authentication requests.

**Salted Hash Generation Workflow:**

1. Combine the salt and the password.
2. Hash the resulting string.

```
echo -n <SALT><PASSWORD> | md5sum
```

## Performance and Constraints

Cracking speed is heavily influenced by the **hashing algorithm** and **hardware**.

|Algorithm|Performance Context (Example)|
|:--|:--|
|**MD5**|High speed (~5 million guesses/sec on a standard laptop).|
|**DCC2**|Low speed (~10,000 guesses/sec on a standard laptop).|

## Wordlist Resources

For dictionary attacks, use established wordlists containing real-world leaked passwords.

- **Location:** `/usr/share/wordlists/rockyou.txt` (common on attack platforms).
- **Viewing Content:**

```
head --lines=20 /usr/share/wordlists/rockyou.txt
```