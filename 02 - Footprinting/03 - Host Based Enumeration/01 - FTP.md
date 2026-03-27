**Basics:**

- Control channel: TCP port 21 (commands)
- Data channel: TCP port 20 (transfers)
- Cleartext protocol — sniffable
- **Active mode**: server connects back to client → blocked by firewalls
- **Passive mode**: client initiates data connection → firewall-friendly

**TFTP** — simpler, uses UDP, no auth, no directory listing, LAN only

---

**Anonymous login:**

```bash
ftp <IP>
# username: anonymous, password: anything
```

**Useful FTP commands:**

```
ls -R        # recursive listing (if enabled)
get <file>   # download file
put <file>   # upload file
debug        # verbose mode
trace        # packet tracing
status       # connection info
```

**Download everything:**

```bash
wget -m --no-passive ftp://anonymous:anonymous@<IP>
```

---

**Nmap enumeration:**

```bash
sudo nmap -sV -p21 -sC -A <IP>
sudo nmap -sV -p21 -sC -A <IP> --script-trace
```

Key NSE scripts:

- `ftp-anon` — checks anonymous access + lists root dir
- `ftp-syst` — runs STAT, shows server version/config
- `ftp-brute` — credential brute force
- `ftp-vsftpd-backdoor` — checks for VSFTPd backdoor

Find all FTP scripts:

```bash
find / -type f -name ftp* 2>/dev/null | grep scripts
```

**Manual interaction:**

```bash
nc -nv <IP> 21
telnet <IP> 21
openssl s_client -connect <IP>:21 -starttls ftp   # TLS + grab cert (hostname, email, org)
```

---

**Dangerous config flags (vsFTPd):**

|Setting|Risk|
|---|---|
|`anonymous_enable=YES`|No creds needed|
|`anon_upload_enable=YES`|Upload → potential RCE via LFI or web server|
|`write_enable=YES`|File write access|
|`hide_ids=YES`|Hides real UIDs — shows "ftp" for everything|
|`ls_recurse_enable=YES`|Full directory tree visible|

**Key files:**

- `/etc/vsftpd.conf` — main config
- `/etc/ftpusers` — users explicitly **denied** FTP access

**Upload = potential RCE** — if FTP is linked to a web server, uploading a webshell → RCE