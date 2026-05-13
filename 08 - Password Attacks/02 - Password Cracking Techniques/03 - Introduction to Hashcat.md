## Hash Type Identification

Unknown hash string identified during exfiltration; requires mapping to a specific numeric ID for processing.

Identify hash type and corresponding mode ID from a raw string.

```
hashid -m '<HASH>'
```

List all supported hash types and IDs directly.

```
hashcat --help
```

- hashID -> `hashid -m '<HASH>'` -> Prefer for quick CLI-based identification.
- Hashcat Example Hashes -> Reference official website -> Prefer for manual identification of obscure or unknown types.

**Incorrect hash mode** — Using the wrong `-m` ID will cause the tool to fail to identify matches even if the correct password is present in the wordlist.

## Straight Wordlist Attack

Standard dictionary-based cracking required when a wordlist is available.

Run basic wordlist attack (Mode 0).

```
hashcat -a 0 -m <PORT> <HASH> <FILE_PATH>
```

**Incorrect hash mode** — Assigning the wrong ID to the hash type results in zero recoveries.

## Rule-Based Attack

Wordlist fails to produce results and common mutations like leetspeak or character appending are necessary to expand the search.

Apply transformations to wordlist entries via ruleset.

```
hashcat -a 0 -m <PORT> <HASH> <FILE_PATH> -r <FILE_PATH>
```

- `best64.rule`: 64 standard modifications including numbers and leet substitutions.
- `leetspeak.rule`: Basic character substitutions.
- `rockyou-30000.rule`: Extensive modifications for larger wordlists.

Standard rule files are located in `/usr/share/hashcat/rules`.

## Mask Attack

Keyspace needs to be restricted to a specific pattern or known length to improve efficiency over raw brute-force.

Execute mask attack (Mode 3) using character set symbols.

```
hashcat -a 3 -m <PORT> <HASH> ?u?l?l?l?d?d
```

- `?l`: abcdefghijklmnopqrstuvwxyz
- `?u`: ABCDEFGHIJKLMNOPQRSTUVWXYZ
- `?d`: 0123456789
- `?s`: Special characters/symbols
- `?a`: Combination of lowercase, uppercase, digits, and symbols

**Broad mask definition** — Defining a mask that covers too large a keyspace on limited hardware will result in unfeasible completion times.