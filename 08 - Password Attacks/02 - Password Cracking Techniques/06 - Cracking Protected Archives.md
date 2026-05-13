1. Identify the container type (ZIP, OpenSSL-encrypted GZIP, BitLocker VHD).
2. For ZIP: Extract the hash using `zip2john` and crack offline with John the Ripper.
3. For OpenSSL-encrypted GZIP: Perform a direct decryption loop using `openssl` to avoid false positives.
4. For BitLocker:
    - Extract hashes using `bitlocker2john`.
    - Isolate the user password hash (`$bitlocker$0`).
    - Crack using Hashcat mode 22100.
5. For Access: Mount decrypted volumes using `dislocker` on Linux or native Explorer on Windows.

---

## ZIP Hash Extraction

Targeting password-protected ZIP archives when native access is restricted and the encryption scheme allows offline hash extraction.

Extract the PKZIP hash for offline cracking

```
zip2john <FILE_PATH> > <HASH_FILE>
```

## ZIP Password Cracking

Executing the offline attack once a hash is successfully formatted.

Run John the Ripper against the extracted ZIP hash

```
john --wordlist=<WORDLIST> <HASH_FILE>
```

Display previously cracked ZIP passwords

```
john <HASH_FILE> --show
```

## OpenSSL Encrypted Archive Brute-forcing

Attempting to decrypt GZIP or TAR archives encrypted via OpenSSL when standalone hash extraction is unreliable.

Brute-force decryption and extraction in a single loop to confirm valid passwords

```
for i in $(cat <WORDLIST>);do openssl enc -aes-256-cbc -d -in <FILE_PATH> -k $i 2>/dev/null| tar xz;done
```

- **False positives** or complete failure to identify the key occur frequently with standard hash cracking; direct extraction confirms the correct password.

## BitLocker Hash Extraction

Targeting Windows full-disk encryption containers (VHD) to obtain crackable material.

Extract all BitLocker-related hashes from a virtual drive

```
bitlocker2john -i <VHD_PATH> > <HASHES_FILE>
```

Filter for the user password hash while ignoring the impractical recovery key hashes

```
grep "bitlocker$0" <HASHES_FILE> > <HASH_FILE>
```

- **Recovery keys** are 48-digit random strings and are not practical for cracking attempts.

## BitLocker Password Cracking

Cracking the isolated AES-encrypted BitLocker user password hash.

Crack the hash using Hashcat with specific BitLocker mode

```
hashcat -a 0 -m 22100 '<HASH>' <WORDLIST>
```

## Mounting Decrypted BitLocker Volumes (Linux)

Accessing the file system of a BitLocker VHD on a Linux attack host after obtaining the cleartext password.

1. Set up the VHD as a loop device

```
sudo losetup -f -P <VHD_PATH>
```

2. Decrypt the specific partition into a mountable dislocker-file

```
sudo dislocker /dev/loop0p2 -u<PASSWORD> -- <MOUNT_POINT_1>
```

3. Mount the decrypted file to the final directory

```
sudo mount -o loop <MOUNT_POINT_1>/dislocker-file <MOUNT_POINT_2>
```

> ⚠️ Gap: Command assumes the target partition is exactly `/dev/loop0p2`; check `lsblk` or `fdisk -l` to verify the correct partition suffix before running `dislocker`.

Unmount volumes after analysis

```
sudo umount <MOUNT_POINT_2>
```

```
sudo umount <MOUNT_POINT_1>
```