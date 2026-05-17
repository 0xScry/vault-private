## Windows File and String Encryption

Handling sensitive data like **NTDS.dit** or credentials when **SSH**, **SFTP**, or **HTTPS** tunnels are unavailable.

Import script as a module before execution

```
Import-Module .\Invoke-AESEncryption.ps1
```

Encrypt string to Base64 ciphertext

```
Invoke-AESEncryption -Mode Encrypt -Key "<PASSWORD>" -Text "Secret Text"
```

Decrypt Base64 string to plain text

```
Invoke-AESEncryption -Mode Decrypt -Key "<PASSWORD>" -Text "<HASH>"
```

Encrypt local file to .aes format

```
Invoke-AESEncryption -Mode Encrypt -Key "<PASSWORD>" -Path <FILE_PATH>
```

Decrypt .aes file

```
Invoke-AESEncryption -Mode Decrypt -Key "<PASSWORD>" -Path <FILE_PATH>
```

**Gotchas**

- **Script dependency** requires transferring `Invoke-AESEncryption.ps1` to the target host before any commands function.
- **Reused passwords** across different client engagements can allow unauthorized decryption if one password is leaked.

---

## Linux File Encryption

Securing high-value files like `/etc/passwd` or enumeration data using **OpenSSL** when secure transport protocols are restricted.

Encrypt file with AES-256 and PBKDF2

```
openssl enc -aes256 -iter 100000 -pbkdf2 -in <FILE_PATH> -out <FILE_PATH>.enc
```

Decrypt encrypted file

```
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in <FILE_PATH>.enc -out <FILE_PATH>
```

**Dangerous / misconfigured settings**

- Missing `-pbkdf2` or low `-iter` counts makes the resulting file vulnerable to **brute-force attacks**.

**Gotchas**

- **In-transit interception** of unencrypted files can lead to severe data leakage; encrypt before moving if using cleartext protocols.

---

## Data Exfiltration Methodology

Symptom-first decision logic for handling sensitive client data.

1. Identify sensitivity: If data contains **PII**, financial records, or trade secrets, do not exfiltrate unless explicitly requested.
2. DLP Testing: Use dummy data that mimics the target format to test **egress filtering** or **DLP controls**.
3. Secure Transport: Prioritize **HTTPS**, **SFTP**, or **SSH** for the transfer itself.
