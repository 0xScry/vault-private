### **Protected File Transfers**

During penetration tests, protecting sensitive data such as **user lists**, **credentials** (e.g., `NTDS.dit`), and **enumeration data** is a professional and legal necessity. Data leakage can have severe consequences for the client and the tester.

#### **Operational Context**

- **When to use:** Use manual encryption when secure transport protocols like **SSH, SFTP, or HTTPS** are unavailable.
- **Why it matters:** Prevents data from being read if intercepted in transit.
- **Best Practice:** Use **strong, unique passwords** for every client to prevent a single password leak from compromising all assessments.
- **DLP Testing:** To test **Data Loss Prevention (DLP)** or egress filtering without exposing actual sensitive data, use **dummy data** that mimics the target information.

---

### **Windows: File Encryption with PowerShell**

The `Invoke-AESEncryption.ps1` script is a lightweight method for encrypting files and strings using **AES**.

#### **Workflow**

1. **Transfer** the script to the target host using any available file transfer method.
2. **Import** the script as a module to make the function available in the current session.
3. **Execute** the encryption or decryption command.

#### **Command Reference**

|Goal|Command|
|:--|:--|
|**Import Module**|`Import-Module .\Invoke-AESEncryption.ps1`|
|**Encrypt String**|`Invoke-AESEncryption -Mode Encrypt -Key "<PASSWORD>" -Text "<SECRET_TEXT>"`|
|**Decrypt String**|`Invoke-AESEncryption -Mode Decrypt -Key "<PASSWORD>" -Text "<CIPHERTEXT>"`|
|**Encrypt File**|`Invoke-AESEncryption -Mode Encrypt -Key "<PASSWORD>" -Path <FILE_PATH>`|
|**Decrypt File**|`Invoke-AESEncryption -Mode Decrypt -Key "<PASSWORD>" -Path <FILE_PATH>.aes`|

_Note: Encrypting a file creates a new file with the **.aes** extension._

---

### **Linux: File Encryption with OpenSSL**

**OpenSSL** is standard on most Linux distributions and is used to encrypt files for secure transit.

#### **Technique Selection**

To resist **brute-force cracking**, it is recommended to use:

- **-aes256**: High-strength cipher.
- **-pbkdf2**: Password-Based Key Derivation Function 2.
- **-iter 100000**: High iteration count to increase the computational cost of cracking.

#### **Operational Commands**

|Goal|Command|
|:--|:--|
|**Encrypt File**|`openssl enc -aes256 -iter 100000 -pbkdf2 -in <INPUT_FILE> -out <OUTPUT_FILE>`|
|**Decrypt File**|`openssl enc -d -aes256 -iter 100000 -pbkdf2 -in <INPUT_FILE> -out <OUTPUT_FILE>`|

**Workflow Details:**

1. When running the encryption command, you will be prompted to **enter and verify the password**.
2. To decrypt, add the `-d` flag and provide the same cipher and iteration parameters used during encryption.
3. Once encrypted, transfer the file using **secure transport** (HTTPS, SFTP, or SSH) whenever possible.