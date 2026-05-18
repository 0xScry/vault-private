## Methodology

1. Identify open SMTP ports 25, 587, or 465
2. Enumerate supported service extensions and commands via EHLO
3. Test for user enumeration via VRFY or EXPN if available
4. Audit for Open Relay misconfigurations using Nmap scripts
5. Attempt manual email transmission to test filtering or spoofing

---

## Service Footprinting

Use when port 25, 587, or 465 shows up in initial port scans.

Enumerate basic service info and supported commands

```
sudo nmap <TARGET_IP> -sC -sV -p<PORT>
```

Initialize connection manually to check banners and EHLO response

```
telnet <TARGET_IP> <PORT>
```

Establish session and list extensions

```
EHLO <DOMAIN>
```

**Tool comparison**

- Nmap
    - `sudo nmap <TARGET_IP> -sC -sV -p25`
    - Prefer for automated command discovery via `smtp-commands` script
- Telnet
    - `telnet <TARGET_IP> 25`
    - Prefer for manual interaction and testing specific protocol responses

**Dangerous / misconfigured settings**

- Standard SMTP (port 25) transmitting **plaintext** authentication or data

**Gotchas** **STARTTLS** is required on port 587 to upgrade to an encrypted connection before authentication.

## User Enumeration

Use when the target SMTP server supports the VRFY or EXPN commands to identify valid system accounts.

Verify a specific username

```
VRFY <USERNAME>
```

**Edge cases**

- If the server is accessed through a web proxy, use the CONNECT method first

```
CONNECT <TARGET_IP>:25 HTTP/1.0
```

**Gotchas** **Response code 252** may be issued by the server to confirm users that do not actually exist.

> ⚠️ Gap: Automated tools may report false positives if the server is configured to return **252** for all queries.

## Manual Interaction and Spoofing

Use when you need to bypass MUAs or test if the MTA/MSA allows spoofed sender addresses.

Complete mail transmission workflow

```
MAIL FROM:<USERNAME>@<DOMAIN>
RCPT TO:<USERNAME>@<DOMAIN>
DATA
From: <USERNAME>@<DOMAIN>
To: <USERNAME>@<DOMAIN>
Subject: <SUBJECT>

<MESSAGE_BODY>
.
```

**Dangerous / misconfigured settings**

- Missing authentication requirements for the **Mail Submission Agent (MSA)**

**Gotchas** The **email header** is separate from the envelope and does not drive technical delivery, but contains info for the recipient.

## Open Relay Identification

Use when an SMTP server is suspected of forwarding mail for unauthorized external senders.

Run automated relay tests

```
sudo nmap <TARGET_IP> -p25 --script smtp-open-relay -v
```

**Dangerous / misconfigured settings**

- `mynetworks = 0.0.0.0/0` in Postfix configuration allows any IP to relay mail

**Gotchas** **16 different tests** are typically performed by Nmap to confirm relay status.