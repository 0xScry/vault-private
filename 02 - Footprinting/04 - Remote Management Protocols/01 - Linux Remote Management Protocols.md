1. Scan for management ports 22, 512-514, and 873.
2. If 22: Run `ssh-audit` to find weak **SSH-1** or vulnerable versions like **OpenSSH 7.2p1** (X11) or **8.2p1** (MITM). Check allowed auth methods manually.
3. If 873: Netcat probe for share names. Attempt anonymous listing. Target `.ssh` directories for lateral movement keys.
4. If 512-514: Attempt `rlogin` using known usernames to hit **.rhosts** or **hosts.equiv** trust bypasses. Use `rwho`/`rusers` for session discovery.

---

## SSH Audit

Port 22 is open and you need to identify weak encryption algorithms or vulnerable server versions.

Audit the service for weak KEX, host-key algorithms, and encryption

```
./ssh-audit.py <TARGET_IP>
```

- **Protocol 1**: Uses outdated encryption and is **vulnerable to MITM attacks**.
- **X11Forwarding yes**: Enables GUI but was **vulnerable to command injection** in OpenSSH 7.2p1.
- **PermitEmptyPasswords yes**: Allows bypass if user has no password set.
- **PermitRootLogin yes**: Allows direct brute-force against the root account.

**SSH-1.99 banner** indicates the server supports both **vulnerable SSH-1** and SSH-2.

## SSH Auth Method Identification

A banner is captured and you need to determine if `password` or `publickey` authentication is accepted before starting a brute-force.

Verbose connection to list supported authentication methods

```
ssh -v <USERNAME>@<TARGET_IP>
```

Force password prompt to verify if password-based logins are enabled

```
ssh -v <USERNAME>@<TARGET_IP> -o PreferredAuthentications=password
```

**CVE-2020-14145** allows MITM attacks during the initial connection attempt on certain versions.

## Rsync Share Discovery

Port 873 is open and you need to identify available modules/shares.

Probe the service to list the internal module names

```
nc -nv <TARGET_IP> 873
```

Query the banner and protocol version

```
sudo nmap -sV -p 873 <TARGET_IP>
```

> ⚠️ Gap: Nmap service scripts or manual `nc` interaction may fail to list shares if the `rsync` daemon is configured with `list = no` in `rsyncd.conf`.

## Rsync Data Exfiltration

A share name is known and you need to pull files or check for sensitive data like SSH keys.

List files in a specific share without downloading

```
rsync -av --list-only rsync://<TARGET_IP>/<SHARE_NAME>
```

Sync all contents from the remote share to the current directory

```
rsync -av rsync://<TARGET_IP>/<SHARE_NAME> .
```

Use when Rsync is configured to tunnel over a non-standard SSH port

```
rsync -av -e "ssh -p <PORT>" rsync://<TARGET_IP>/<SHARE_NAME>
```

**Password re-use** is common; use credentials found elsewhere if anonymous access fails.

## R-Services Authentication Bypass

Ports 512, 513, or 514 are open and you are testing for trusted host relationships.

Attempt login without a password using a specific username

```
rlogin <TARGET_IP> -l <USERNAME>
```

Execute a single command on the remote host via rexec (requires credentials)

```
rexec -p <PASSWORD> -l <USERNAME> <TARGET_IP> <SERVICE_NAME>
```

- **+ Wildcard**: A `+` in `/etc/hosts.equiv` or `.rhosts` allows **any user from any host** to log in without a password.
- **Unencrypted Traffic**: All R-services transmit data in cleartext and are **vulnerable to MITM**.

**Authentication fails** if the source IP/hostname and username do not exactly match an entry in the remote `.rhosts` or `/etc/hosts.equiv` files.

## R-Services User Enumeration

You have established a shell or have network access and need to scope out active users for further targeting.

List all users currently logged into the local network

```
rwho
```

Detailed list of logged-in users including TTY and login time

```
rusers -al <TARGET_IP>
```

**rwho relies on broadcasts**; if the daemon is not actively broadcasting, the command will return no results even if users are logged in.