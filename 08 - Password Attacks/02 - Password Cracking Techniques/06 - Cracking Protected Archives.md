# Cracking Protected Archives

Identifying and accessing password-protected archives is a critical step in data exfiltration and credential harvesting. While many formats like **ZIP** and **7z** support native encryption, others like **TAR** often rely on external tools like **OpenSSL** or **GPG** for protection.

## Archive Identification

Before attempting to crack an archive, determine the actual file type. Extensions can be misleading, especially when external encryption is used.

|Command|Purpose|
|:--|:--|
|`file <FILE>`|Identifies the file type and indicates if it is **OpenSSL encrypted**|
|`curl -s https://fileinfo.com/filetypes/compressed \|html2text \|

---

## Cracking ZIP Archives

**ZIP** files are frequently encountered in **Windows environments**. To crack them, you must first extract the password hash for offline processing.

### Operational Workflow

1. **Extract the hash:** Use `zip2john` to format the archive's protection into a crackable string.
2. **Run the cracker:** Use **John the Ripper (JtR)** with a wordlist against the extracted hash.
3. **View the password:** Once cracked, use the `--show` flag to retrieve the cleartext password.

### Command Reference

|Command|Description|
|:--|:--|
|`zip2john <ARCHIVE>.zip > <HASH_FILE>`|Extracts the hash from a ZIP archive|
|`john --wordlist=<PASSWORD_LIST> <HASH_FILE>`|Initiates the offline brute-force/dictionary attack|
|`john <HASH_FILE> --show`|Displays the cracked password|

---

## Cracking OpenSSL Encrypted GZIP Files

When `file` identifies a GZIP archive as **"openssl enc'd data with salted password,"** traditional hash extraction may be unreliable due to **false positives**.

### Operational Workflow

Instead of extracting a hash, use a **bash for loop** to attempt direct decryption. This method is more reliable as it succeeds only when the correct password successfully extracts the archive contents.

1. **Execute the decryption loop:** Iterate through a wordlist, passing each string to `openssl`.
2. **Verify extraction:** Check the current directory for newly created files.

### Command Reference

|Command|Description|
|:--|:--|
|`for i in $(cat <PASSWORD_LIST>); do openssl enc -aes-256-cbc -d -in .gzip -k $i 2>/dev/null \|tar xz; done`|

---

## Cracking BitLocker-Encrypted Drives

**BitLocker** is Microsoft's full-disk encryption. In enterprise environments, it is often used for **virtual drives (.vhd)** containing sensitive notes or documents.

### Operational Workflow

1. **Extract hashes:** Use `bitlocker2john` to identify the protection types (Password and Recovery Key).
2. **Isolate the password hash:** Focus on the `$bitlocker$0` hash. The **Recovery Key** (a 48-digit string) is generally impractical to crack without partial knowledge.
3. **Crack with Hashcat:** Use **Mode 22100** for BitLocker hashes. Note that AES encryption is computationally expensive and may take significant time.

### Command Reference

|Command|Description|
|:--|:--|
|`bitlocker2john -i <DRIVE>.vhd > <HASH_FILE>`|Extracts BitLocker hashes from a virtual drive|
|`grep "bitlocker$0" <HASH_FILE> > <TARGET_HASH>`|Isolates the user password hash for cracking|
|`hashcat -a 0 -m 22100 '<HASH>' <PASSWORD_LIST>`|Cracks the BitLocker hash using Hashcat|

---

## Mounting BitLocker Drives (Linux)

After successfully cracking the password, you must mount the drive to access the contents.

### Operational Workflow

1. **Setup loop device:** Use `losetup` to present the `.vhd` as a block device.
2. **Decrypt the volume:** Use `dislocker` with the cracked password to create a decrypted file.
3. **Mount the filesystem:** Mount the `dislocker-file` to a local directory.

### Command Reference

| Command                                                             | Description                              |
| :------------------------------------------------------------------ | :--------------------------------------- |
| `sudo losetup -f -P <DRIVE>.vhd`                                    | Configures the VHD as a loop device      |
| `sudo dislocker /dev/loop0p2 -u<PASSWORD> -- <MOUNT_POINT_1>`       | Decrypts the BitLocker drive             |
| `sudo mount -o loop <MOUNT_POINT_1>/dislocker-file <MOUNT_POINT_2>` | Mounts the decrypted volume for browsing |
| `sudo umount <MOUNT_POINT_1> <MOUNT_POINT_2>`                       | Unmounts the volumes after analysis      |