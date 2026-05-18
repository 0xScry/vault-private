1. Scan for TCP **port 21** or UDP **port 69** to identify file transfer services.
2. Attempt **anonymous login** on FTP using `anonymous` or `ftp` with no password.
3. If the connection fails or hangs, switch between **active and passive FTP** modes to bypass firewalls.
4. For encrypted connections, use OpenSSL to extract certificates and potential **hostnames or email addresses**.
5. On TFTP, proceed blindly with `get` for common configuration files since **directory listing is unavailable**.
6. Audit `/etc/vsftpd.conf` if local access is gained to find **misconfigured upload permissions**.

---

## FTP Interaction and Enumeration

Target exposes TCP port 21 and potentially allows unauthenticated access.

Interactive login attempt

```
ftp <TARGET_IP>
```

Grab version and status information

```
ftp> status
```

Enable detailed packet output for troubleshooting data channel issues

```
ftp> debug
ftp> trace
```

Recursive directory listing to map the entire filesystem at once

```
ftp> ls -R
```

- **Tool comparison**
    
    - `ftp`: standard interactive client; use for manual exploration
    - `nc`: `nc -nv <TARGET_IP> 21`; use for raw banner grabbing
    - `telnet`: `telnet <TARGET_IP> 21`; alternative for manual banner interaction
- **Dangerous / misconfigured settings**
    
    - `anonymous_enable=YES`: allows public access without credentials
    - `no_anon_password=YES`: bypasses password prompt for anonymous users
    - `write_enable=YES`: enables commands like STOR, DELE, and MKD
- **Gotchas**
    
    - **Firewall blocks responses** in active mode; use `PASV` command to force the server to provide a data port.
    - **Clear-text protocol** allows credentials to be sniffed if the network environment permits.

## FTP Mass Exfiltration

Directory listing is successful and contains a large volume of data or deep nesting.

Mirror the entire FTP share locally without interactive prompts

```
wget -m --no-passive ftp://anonymous:anonymous@<TARGET_IP>
```

- **Edge cases**
    - Mirroring large shares can **trigger alerts** due to unusual traffic volume.

## Encrypted FTP Discovery

Service requires TLS/SSL or you need to extract metadata from the certificate.

Connect using STARTTLS to view the certificate chain

```
openssl s_client -connect <TARGET_IP>:21 -starttls ftp
```

- **Edge cases**
    - SSL certificates often contain **hostnames, internal locations, or admin emails** in the CN or OU fields.

## Automated FTP Footprinting

Rapid identification of vulnerabilities or configuration flaws across multiple targets.

Update the NSE database before scanning

```
sudo nmap --script-updatedb
```

Run default FTP scripts and version detection

```
sudo nmap -sV -p21 -sC -A <TARGET_IP>
```

Trace script execution to see raw commands like `STAT` or `PORT`

```
sudo nmap -sV -p21 -sC -A <TARGET_IP> --script-trace
```

- **Tool comparison**
    - `ftp-anon.nse`: checks for anonymous access and lists root directory
    - `ftp-syst.nse`: runs `STAT` to find server status and version

## TFTP Interaction

UDP port 69 is open; requires blind file retrieval due to lack of authentication and listing features.

Connect to the remote host

```
tftp <TARGET_IP>
```

Download a specific file

```
tftp> get <FILE_PATH>
```

- **Dangerous / misconfigured settings**
    
    - **Global read/write permissions** are often required on the OS level for TFTP to function, exposing sensitive files.
- **Gotchas**
    
    - **Unreliable protocol** because it uses UDP; requires application-layer recovery.
    - > ⚠️ Gap: TFTP lacks directory listing; you must have a pre-existing list of filenames or guess them to retrieve data.
        

## VSFTPD Configuration Audit

Local access or a misconfigured upload allows reading the server configuration file.

Review active configuration settings excluding comments

```
cat /etc/vsftpd.conf | grep -v "#"
```

Check for users explicitly denied FTP access

```
cat /etc/ftpusers
```

- **Dangerous / misconfigured settings**
    
    - `anon_upload_enable=YES`: allows anonymous users to upload files
    - `anon_mkdir_write_enable=YES`: allows anonymous users to create directories
    - `chroot_local_user=NO`: may allow users to escape their home directory if not enabled
- **Gotchas**
    
    - **Hiding IDs** via `hide_ids=YES` will show all file owners as `ftp`, masking the actual UIDs/GIDs.