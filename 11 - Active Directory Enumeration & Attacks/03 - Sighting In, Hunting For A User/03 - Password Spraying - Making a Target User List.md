## SMB NULL Session User Enumeration

When to use — Internal network access without credentials and SMB NULL sessions are available on the Domain Controller.

Commands — Use enum4linux to retrieve and parse users into a clean list.

```
enum4linux -U <TARGET_IP> | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

Manual enumeration via rpcclient when custom queries are needed.

```
rpcclient -U "" -N <TARGET_IP>
```

```
enumdomusers
```

Identify users and their **badpwdcount** to avoid locking accounts during a spray.

```
crackmapexec smb <TARGET_IP> --users
```

Tool comparison

- enum4linux -> `enum4linux -U <TARGET_IP>` -> prefer for quick parsing of large user lists
- rpcclient -> `rpcclient -U "" -N <TARGET_IP>` -> prefer for interactive manual inspection
- CrackMapExec -> `crackmapexec smb <TARGET_IP> --users` -> prefer for identifying lockout proximity via **badpwdcount**

Edge cases — In multi-DC environments, **badpwdcount** and **badpwdtime** are maintained separately per DC; query the **PDC Emulator** for accurate totals.

Gotchas — **Account lockout** occurs if spraying without checking the badpwdcount against the domain policy.

---

## LDAP Anonymous Bind User Enumeration

When to use — Anonymous binds are enabled on the DC LDAP service.

Commands — Use ldapsearch with custom filters for raw attribute extraction.

```
ldapsearch -h <TARGET_IP> -x -b "DC=<DOMAIN>,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "
```

Automate user extraction with windapsearch.

```
./windapsearch.py --dc-ip <DC_IP> -u "" -U
```

Tool comparison

- windapsearch -> `./windapsearch.py --dc-ip <DC_IP> -u "" -U` -> prefer for automated, clean output
- ldapsearch -> `ldapsearch -h <TARGET_IP> -x ...` -> prefer when custom LDAP filters are required

> ⚠️ Gap: Source mentions password policy can be enumerated via anonymous bind but does not provide the specific tool commands or flags to do so.

---

## Kerberos Username Enumeration

When to use — No session access is available but Port 88 is reachable; requires speed and avoids logon failure events.

Commands — Enumerate users using a wordlist via Kerberos Pre-Authentication.

```
kerbrute userenum -d <DOMAIN> --dc <DC_IP> <FILE_PATH>
```

Tool comparison

- Kerbrute -> `kerbrute userenum` -> prefer for speed and avoiding **Event ID 4625** (logon failure)

Dangerous / misconfigured settings

- Kerberos event logging enabled via Group Policy triggers **Event ID 4768** on TGT requests.

Gotchas — **PRINCIPAL UNKNOWN** error indicates an invalid username, while a prompt for Pre-Authentication confirms the user exists.

---

## Credentialed User Enumeration

When to use — Valid domain credentials exist or SYSTEM access is obtained on a domain-joined host.

Commands — Leverage existing credentials to dump the full user list and login statistics.

```
crackmapexec smb <TARGET_IP> -u <USERNAME> -p <PASSWORD> --users
```

Edge cases — SYSTEM accounts can be used for enumeration because they impersonate the computer object, which functions as a domain user.

Gotchas — **Account lockout** remains a risk if automated tools attempt authentication with expired or incorrect passwords during enumeration.

---

## External Resource Enumeration

When to use — Internal enumeration via SMB, LDAP, and Kerberos is unsuccessful.

Commands — Use LinkedIn to generate potential username lists.

```
linkedin2username
```

Edge cases — User lists from external sources like email harvesting or LinkedIn are often incomplete compared to AD queries.