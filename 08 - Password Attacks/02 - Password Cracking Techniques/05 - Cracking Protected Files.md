# Cracking Protected Files

## Hunting for Encrypted Files and SSH Keys

During post-exploitation, identifying encrypted files is critical as they often contain **sensitive data** or **credentials** required for further lateral movement.

### Locating Files by Extension

Use this technique to identify common office documents and archives that likely contain sensitive professional or personal data.

1. **Execute a recursive search** for common encrypted file extensions while filtering out noise from system directories.

```
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*"); do
    echo -e "\nFile extension: " $ext;
    find / -name *$ext 2>/dev/null | grep -v "lib|fonts|share|core";
done
```

### Locating SSH Keys by Content

SSH keys may not have standard extensions. Search for unique **header strings** to identify private keys.

1. **Search recursively** for the standard SSH private key header.

```
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /<DIRECTORY> 2>/dev/null
```

---

## Identifying Encrypted SSH Keys

Before attempting to use a discovered SSH key, you must determine if it is **passphrase-protected**.

|Method|Indicator|Context|
|:--|:--|:--|
|**Header Check**|`Proc-Type: 4,ENCRYPTED`|Works for older **PEM formats**.|
|**ssh-keygen**|Prompts for a passphrase|Used for **modern keys** that appear identical whether encrypted or not.|

### Validation Workflow

1. **Attempt to read the key** using `ssh-keygen`.
2. **Analyze the output**: If it asks for a passphrase, the file must be cracked before use.

```
ssh-keygen -yf <PRIVATE_KEY_PATH>
```

---

## Extraction and Cracking Workflow

To crack protected files, you must first **extract the hash** into a format John the Ripper (JtR) understands.

### 1. Locate Conversion Scripts

JtR includes various "2john" scripts to handle different file types.

```
locate *2john*
```

### 2. Extract and Crack Hashes

Follow this sequence to convert the encrypted file into a crackable hash and then perform an **offline brute-force attack**.

|Target File Type|Extraction Tool|Attack Implication|
|:--|:--|:--|
|**SSH Private Key**|`ssh2john.py`|Unlocks remote access to the target host.|
|**MS Office Doc**|`office2john.py`|Reveals protected internal documentation.|
|**PDF Document**|`pdf2john.py`|Accesses intercepted sensitive reports or contracts.|

#### Operational Steps:

1. **Extract the hash** from the protected file and redirect it to a file.
2. **Run the cracker** against the hash using a wordlist (e.g., `rockyou.txt`).
3. **Display the result** once the crack is successful.

**SSH Key Cracking:**

```
ssh2john.py <PRIVATE_KEY> > <OUTPUT_HASH_FILE>
john --wordlist=<WORDLIST_PATH> <OUTPUT_HASH_FILE>
john <OUTPUT_HASH_FILE> --show
```

**Microsoft Office Cracking:**

```
office2john.py <OFFICE_FILE> > <OUTPUT_HASH_FILE>
john --wordlist=<WORDLIST_PATH> <OUTPUT_HASH_FILE>
john <OUTPUT_HASH_FILE> --show
```

**PDF Cracking:**

```
pdf2john.py <PDF_FILE> > <OUTPUT_HASH_FILE>
john --wordlist=<WORDLIST_PATH> <OUTPUT_HASH_FILE>
john <OUTPUT_HASH_FILE> --show
```

---

## Operational Considerations

- **Algorithm Types**: Most local files use **symmetric encryption** (e.g., AES-256), where the same key is used for encryption and decryption.
- **Success Factors**: Success depends heavily on the quality of the **wordlist**. Standard lists may fail against complex, randomly generated passphrases.
- **False Positives**: The SSH cracking format in JtR may emit **false positives**; the tool will continue trying even after a potential candidate is found.