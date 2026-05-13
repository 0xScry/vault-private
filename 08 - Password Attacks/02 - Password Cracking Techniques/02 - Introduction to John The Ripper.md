## Hash Identification

Symptom: Hash format is unknown or JtR fails to identify it automatically.

Check hash against built-in lists to get potential JtR-specific format names

```
hashid -j <HASH>
```

Identify format by looking up sample hashes or known service contexts

```
grep <HASH_TYPE> /usr/share/john/john.conf
```

- **Tool comparison**
    
    - hashid -> `hashid -j <HASH>` -> prefer for quick identification of common formats with JtR flag support
    - Manual Lookup -> check PentestMonkey or JtR sample docs -> prefer when tool output is ambiguous or multiple formats are suggested
- **Gotchas**
    
    - **Ambiguous identification** occurs when hash context is missing, requiring manual testing of suggested formats.

---

## File to Hash Conversion

Symptom: Target is an encrypted file, private key, or database rather than a raw hash string.

Convert encrypted file to a format JtR can parse

```
<SERVICE_NAME>2john <FILE_PATH> > <HASH_FILE>
```

Locate all available conversion scripts on the system

```
locate *2john*
```

- **Edge cases**
    - Script location: some conversion tools are in `/usr/bin/` while others are `.py` or `.pl` scripts in `/usr/share/john/`.

---

## Single Crack Mode

Symptom: Targeting Linux credentials where username, home directory, or GECOS information is known.

Run rule-based attack using local account information to generate candidates

```
john --single <FILE_PATH>
```

- **Gotchas**
    - **Missing account metadata** in the input file will result in zero candidates being generated for this mode.

---

## Wordlist Mode

Symptom: Dictionary attack required against a known or custom list of potential passwords.

Run standard dictionary attack

```
john --wordlist=<FILE_PATH> <HASH_FILE>
```

Apply common mutations like capitalization or trailing numbers during wordlist processing

```
john --wordlist=<FILE_PATH> --rules <HASH_FILE>
```

Combine multiple wordlists for a single session

```
john --wordlist=<FILE_PATH_1>,<FILE_PATH_2> <HASH_FILE>
```

---

## Incremental Mode

Symptom: Wordlists are exhausted and exhaustive brute-force is required using statistical likelihood.

Run brute-force attack using default character sets and lengths

```
john --incremental <HASH_FILE>
```

Use a specific character set defined in the configuration file

```
john --incremental=<SERVICE_NAME> <HASH_FILE>
```

- **Dangerous / misconfigured settings**
    
    - Large character sets or long MaxLen values in `john.conf` will make attacks effectively infinite.
- **Gotchas**
    
    - **High resource consumption** makes this mode significantly slower than wordlist or single modes.

---

## Manual Format Override

Symptom: JtR fails to recognize the hash or you need to force a specific protocol parser.

Force JtR to use a specific hash format parser

```
john --format=<SERVICE_NAME> <HASH_FILE>
```

---

## Viewing Results

Symptom: Need to retrieve cracked credentials from a previous or ongoing session.

Display cracked passwords reliably from the JtR database

```
john --show <HASH_FILE>
```