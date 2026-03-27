# John The Ripper (JtR)

John the Ripper is an open-source tool used for cracking passwords through brute-force and dictionary attacks. The **"jumbo" variant** is the industry standard due to performance optimizations, multilingual wordlist support, and 64-bit architecture compatibility.

### Hash Identification

Before cracking, you must identify the hash format. While JtR often auto-detects, manual identification is required for unknown formats.

1. **Analyze the context** of where the hash was found to make an educated guess.
2. **Use hashID** with the `-j` flag to map hashes to specific JtR format strings.

|Command|Purpose|
|:--|:--|
|`hashid -j <HASH>`|Identifies hash type and provides the specific JtR format name.|

---

### Cracking Modes

#### 1. Single Crack Mode

- **When to use:** Use as a first step when targeting Linux credentials or when user metadata (username, GECOS, home directory) is available.
- **Why it matters:** It is highly efficient because it generates candidates based on the victim's specific information (e.g., real name, phone number) and applies common string modifications.
- **Attack Implication:** This can quickly crack passwords that are variations of personal details.

|Command|Parameters|
|:--|:--|
|`john --single <HASH_FILE>`|Executes a rule-based attack using account metadata.|

#### 2. Wordlist Mode

- **When to use:** Use when you have a dictionary of potential passwords.
- **Why it matters:** This is a standard dictionary attack that tests every entry in a text file against the hash.
- **Decision Point:** Apply **--rules** to the wordlist to generate transformations (capitalization, special characters, numbers), increasing the chance of success without requiring a larger wordlist.

|Command|Parameters|
|:--|:--|
|`john --wordlist=<WORDLIST_FILE> <HASH_FILE>`|Standard dictionary attack.|
|`john --wordlist=<WL1>,<WL2> <HASH_FILE>`|Runs multiple wordlists sequentially.|
|`john --wordlist=<WORDLIST_FILE> --rules <HASH_FILE>`|Applies transformations to wordlist entries.|

#### 3. Incremental Mode

- **When to use:** Use as a last resort when wordlists fail.
- **Why it matters:** This is an exhaustive brute-force mode using **Markov chains** (statistical models) to prioritize likely character combinations.
- **Edge Case:** This mode is resource-intensive and slow for long or complex passwords; performance can be improved by defining specific character sets or lengths in `john.conf`.

|Command|Parameters|
|:--|:--|
|`john --incremental <HASH_FILE>`|Starts exhaustive brute-force using default character sets.|
|`john --incremental=<CHARSET> <HASH_FILE>`|Focuses brute-force on a specific character set (e.g., ASCII, Latin1).|

---

### Operating on Encrypted Files

JtR cannot crack encrypted files (PDFs, ZIPs, SSH keys) directly. You must first extract the hash using a **"2john" utility**.

**Operational Workflow:**

1. Locate the appropriate conversion tool (e.g., `ssh2john`, `zip2john`).
2. Convert the file into a JtR-readable hash format and redirect output to a file.
3. Run JtR against the generated hash file.

|Command|Purpose|
|:--|:--|
|`<TOOL> <FILE_TO_CRACK> > <HASH_FILE>`|Converts an encrypted file into a crackable hash.|
|`locate *2john*`|Lists all available conversion utilities on the system.|

**Common Conversion Tools:**

|Tool|Target File Type|
|:--|:--|
|`pdf2john`|PDF documents|
|`ssh2john`|SSH private keys|
|`zip2john`|ZIP archives|
|`rar2john`|RAR archives|
|`keepass2john`|KeePass databases|
|`office2john`|MS Office documents|

---

### Command Reference: Specific Hash Formats

Use the `--format=` argument if JtR fails to identify the hash or if multiple candidates exist.

|Format String|Description|
|:--|:--|
|`nt`|Windows NT/NTLM hashes|
|`netntlmv2`|Network NTLMv2 hashes|
|`raw-md5` / `raw-sha1`|Standard raw MD5 or SHA1 hashes|
|`mssql` / `mysql`|Database-specific password hashes|
|`descrypt` / `sha512crypt`|Common Unix/Linux password hashes|
|`openssha`|OpenSSH private key passwords|