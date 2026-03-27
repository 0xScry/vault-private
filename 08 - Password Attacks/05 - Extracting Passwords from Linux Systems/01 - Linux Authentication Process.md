### Linux Authentication & Credential Harvesting

The Linux authentication process is primarily managed by **Pluggable Authentication Modules (PAM)**, specifically `pam_unix.so` or `pam_unix2.so`. These modules utilize standardized API calls to manage user information, sessions, and password updates by interacting with specific system files.

#### Critical System Files

|File|Purpose|Permissions|
|:--|:--|:--|
|`/etc/passwd`|Stores user account information (7 structured fields).|World-readable.|
|`/etc/shadow`|Stores hashed passwords and password management data.|Admin (root) only.|
|`/etc/security/opasswd`|Stores previous passwords to prevent reuse.|Admin (root) only.|

---

#### 1. Exploiting Misconfigured `/etc/passwd` Permissions

**Scenario:** Use when the `/etc/passwd` file is **writeable** by a low-privileged user due to administrative error.

**Goal:** Bypass authentication by removing the root password requirement.

1. **Identify write permissions** on `/etc/passwd`.
2. **Modify the root entry** to remove the 'x' (or any value) in the password field.
    
    ```
    # Original entry: root:x:0:0:root:/root:/bin/bash
    # Modified entry:
    root::0:0:root:/root:/bin/bash
    ```
    
3. **Switch to the root user**; the system will no longer prompt for a password.
    
    ```
    su
    ```
    

---

#### 2. Identifying Password Hashes

When analyzing `/etc/shadow` or old passwords in `opasswd`, the **ID value** in the hash indicates the cryptographic algorithm used. Identifying the algorithm is critical for selecting the correct **cracking mode** and estimating the time required.

|ID|Cryptographic Hash Algorithm|Note|
|:--|:--|:--|
|`1`|MD5|Easier to crack; useful for pattern recognition.|
|`2a`|Blowfish|Older algorithm.|
|`5`|SHA-256|Standard algorithm.|
|`6`|SHA-512|Standard algorithm.|
|`y`|Yescrypt|Modern default for many distributions like Debian.|
|`sha1`|SHA1crypt|Older algorithm.|

---

#### 3. Credential Harvesting & Cracking Workflow

**Scenario:** Use once **root access** is obtained to recover plaintext passwords for other users or to identify password reuse patterns across the network.

**Operational Steps:**

1. **Backup the target files** to a temporary directory to avoid accidental corruption.
    
    ```
    sudo cp /etc/passwd /tmp/passwd.bak
    sudo cp /etc/shadow /tmp/shadow.bak
    ```
    
2. **Combine the files** into a format readable by cracking tools using `unshadow`.
    
    ```
    unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
    ```
    
3. **Perform the crack** using `hashcat` or John the Ripper (JtR).
    
    ```
    # Using Hashcat with a wordlist (Mode 1800 for SHA-512)
    hashcat -m <MODE_ID> -a 0 /tmp/unshadowed.hashes <WORDLIST_PATH> -o /tmp/unshadowed.cracked
    ```
    

**Attack Implications:**

- **Pattern Recognition:** Recovering old hashes from `/etc/security/opasswd` helps identify if users follow predictable patterns when updating passwords, increasing the success rate of password guessing.
- **Disabled Logins:** If the password field in `/etc/shadow` contains `!` or `*`, Unix password login is disabled, though **key-based** or **Kerberos** authentication may still be active.