1. Capture hash and identify algorithm (**MD5**, **SHA-256**).
2. Check for **salting**; if unsalted, prioritize **rainbow tables** for near-instant identification.
3. If salted or no table match, execute a **dictionary attack** using `rockyou.txt` or **SecLists**.
4. If dictionary fails and password length is estimated <9 characters, initiate a **brute-force attack**.
5. Adjust expectations based on algorithm; **MD5** allows high-speed guessing, while **DCC2** significantly throttles attempts.

---

## Manual Hash Generation

Verification of cleartext candidates against captured hashes.

Verify a suspected cleartext matches an **MD5** hash

```
echo -n <PASSWORD> | md5sum
```

Verify a suspected cleartext matches a **SHA-256** hash

```
echo -n <PASSWORD> | sha256sum
```

Generate a hash with a known prepended salt

```
echo -n <SALT><PASSWORD> | md5sum
```

**Gotchas** **Missing -n flag** in `echo` will include a trailing newline in the input, resulting in an incorrect hash value.

## Rainbow Table Attacks

Captured hash is unsalted and exists in pre-compiled mappings.

**Dangerous / misconfigured settings**

- **Lack of salting** allows attackers to bypass computational requirements via pre-compiled input/output maps.

**Gotchas** **Salting** invalidates rainbow tables by requiring attackers to re-map the entire table for every possible salt value.

## Dictionary Attacks

Default technique for time-constrained engagements using statistical likelihood.

Preview the first 20 entries of a wordlist to verify format

```
head --lines=20 <FILE_PATH>
```

**Edge cases**

- **rockyou.txt** contains over 14 million real-world passwords and is the standard starting point for most assessments.

**Gotchas** **Incomplete wordlists** will fail to identify any password not present in the specific statistical set being used.

## Brute-Force Attacks

Exhaustive character combination attempts when dictionary attacks fail.

**When to use** Last resort for **short passwords** (<9 characters) where 100% effectiveness is required despite time costs.

**Tool comparison**

- **hashcat**
    - `hashcat`
    - Prefer for high-speed cracking on local hardware; performance varies by algorithm.

**Gotchas** **High-iteration algorithms** like **DCC2** drastically reduce cracking speeds compared to simple algorithms like **MD5**, making brute-force non-viable.

> ⚠️ Gap: Source lacks specific **hashcat** command syntax for executing dictionary or brute-force attacks; only mentions the tool's performance capabilities. ⚠️ Gap: Source mentions **mask attacks** as a more efficient alternative to brute-force but provides no implementation details.