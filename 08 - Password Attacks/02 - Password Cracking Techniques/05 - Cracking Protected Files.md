## Encrypted File Discovery

When to use: Post-exploitation data gathering on Linux targets when searching for sensitive documentation or lateral movement vectors.

Recursive search for common Office, PDF, and OpenDocument extensions:

```
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: "  $ext; find / -name *$ ext 2>/dev/null | grep -v "lib|fonts|share|core" ;done
```

Recursive content-based search for SSH private keys using standard header/footer values:

```
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$ ' / 2>/dev/null
```

,

## SSH Key Identification

When to use: Found a potential private key and need to verify if it is **encrypted** or valid before pivot attempts.

Validate key and check for passphrase requirement using ssh-keygen:

```
ssh-keygen -yf <FILE_PATH>
```

Output indicating a **password-protected** key:

```
Enter passphrase for "<FILE_PATH>":
```

Gotchas:

- **Modern SSH keys** look identical in the header whether encrypted or not; older PEM formats explicitly list encryption methods in the header.

## SSH Key Cracking

When to use: A found SSH private key requires a passphrase for use.

Locate required extraction scripts on the attack box:

```
locate *2john*
```

Extract hash and crack via John the Ripper (JtR):

```
ssh2john.py <FILE_PATH> > <HASH>
john --wordlist=rockyou.txt <HASH>
```

View the cracked passphrase:

```
john <HASH> --show
```

Gotchas:

- The SSH cracking format in JtR may emit **false positives**, causing the tool to continue searching after finding a potential candidate.

## Office and PDF Document Cracking

When to use: Encountering locked .docx, .xlsx, or .pdf files during data exfiltration,.

Extract and crack Microsoft Office hashes:

```
office2john.py <FILE_PATH> > <HASH>
john --wordlist=rockyou.txt <HASH>
```

Extract and crack PDF hashes:

```
pdf2john.py <FILE_PATH> > <HASH>
john --wordlist=rockyou.txt <HASH>
```

Tool comparison:

- **John the Ripper** -> `john <HASH>` -> Prefer for initial extraction using included Python scripts.
- **Hashcat** -> `hashcat <HASH>` -> Use for high-performance cracking once the hash is extracted.

Gotchas:

- **Security mechanisms** or long, randomly generated passphrases may prevent cracking within engagement timeframes.