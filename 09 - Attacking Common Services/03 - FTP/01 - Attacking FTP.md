## Enumeration

Port 21 is open and requires service versioning and script-based vulnerability identification.

Scan for version banners and anonymous login status

```
sudo nmap -sC -sV -p 21 <TARGET_IP>
```

Interactive service banner grabbing

```
nc -nv <TARGET_IP> 21
```

- **Tool comparison**
    - Nmap -> `nmap -sC -sV` -> Use for automated script execution and version detection
    - Netcat -> `nc -nv` -> Use for manual banner verification if Nmap results are inconclusive

**Gotchas** **Truncated directory listings** in Nmap output require `--script-args ftp-anon.maxlist=-1` to view all files.

## Anonymous Login

`ftp-anon` script returns success or manual connection accepts **anonymous** as a username.

Connect to service and authenticate without a password

```
ftp <TARGET_IP>
```

Download a single file

```
get <FILE_PATH>
```

Download multiple files from the current directory

```
mget *
```

Upload a local file to the server

```
put <FILE_PATH>
```

- **Dangerous / misconfigured settings**
    - Read/write permissions enabled for the anonymous user
    - Sensitive files stored in the FTP root or subdirectories

**Gotchas** **Write permissions** combined with web-accessible directories may allow for RCE via uploaded scripts if the file path can be identified.

## Brute Forcing

Anonymous authentication is disabled and a list of potential users or passwords is available.

Targeted brute force against a specific user

```
medusa -u <USERNAME> -P <FILE_PATH> -h <TARGET_IP> -M ftp
```

Credential list-based attack

```
medusa -U <FILE_PATH> -P <FILE_PATH> -h <TARGET_IP> -M ftp
```

- **Tool comparison**
    - Medusa -> `medusa -M ftp` -> Use for high-speed automated authentication attempts

**Gotchas** **Account lockout mechanisms** usually prevent successful brute forcing on modern applications; password spraying is more effective.

## FTP Bounce Attack

An exposed FTP server can reach internal network segments that are not directly accessible from the attack host.

Scan internal targets through the FTP proxy

```
nmap -Pn -v -n -p <PORT> -b <USERNAME>:<PASSWORD>@<PIVOT_IP> <TARGET_IP>
```

- **Dangerous / misconfigured settings**
    - FTP `PORT` command allowed to connect to arbitrary internal IP addresses

**Gotchas** **Modern FTP protections** typically disable the bounce attack capability by default.