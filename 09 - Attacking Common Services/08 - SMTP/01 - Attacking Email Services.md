# Attacking Email Services

## Initial Reconnaissance & Infrastructure Identification

The first step in attacking email services is identifying the mail server responsible for a domain to determine if the target uses a **cloud-based provider** (e.g., Microsoft 365, G-Suite) or a **custom implementation**. This distinction is critical because cloud providers use modern authentication and unique attack vectors, while custom servers may contain protocol-level misconfigurations.

### 1. Identify Mail Exchanger (MX) Records

Use DNS queries to locate the mail servers for a domain.

|Tool|Command|Goal|
|:--|:--|:--|
|`host`|`host -t MX <DOMAIN>`|Identify the mail server handling the domain.|
|`dig`|`dig mx \|grep "MX" \|
|`host`|`host -t A <MAIL_SERVER_HOSTNAME>`|Resolve the mail server's IP address.|

### 2. Service Scanning

Once the IP is identified, perform a targeted port scan to find active email protocols.

|Port|Service|Encryption|
|:--|:--|:--|
|**TCP/25**|SMTP|Unencrypted|
|**TCP/465**|SMTP|Encrypted|
|**TCP/587**|SMTP|Encrypted/STARTTLS|
|**TCP/110**|POP3|Unencrypted|
|**TCP/995**|POP3|Encrypted|
|**TCP/143**|IMAP4|Unencrypted|
|**TCP/993**|IMAP4|Encrypted|

**Operational Command:**

```
sudo nmap -Pn -sV -sC -p25,110,143,465,587,993,995 <TARGET_IP>
```

_Note: Use `-sC` to run default Nmap scripts for service enumeration._

---

## User Enumeration

Enumerating valid usernames allows for subsequent password spraying or brute-force attacks.

### Technique 1: SMTP Manual Interaction

Use `telnet` to connect to port 25 and test specific SMTP commands.

|Command|Purpose|
|:--|:--|
|`VRFY`|Checks the validity of a specific username.|
|`EXPN`|Expands distribution lists to reveal individual user emails.|
|`RCPT TO`|Identifies recipients during the email sending process; indicates if a user exists.|

**Workflow for `VRFY`:**

1. Connect to the server: `telnet <TARGET_IP> 25`.
2. Issue the command: `VRFY <USERNAME>`.
3. **Decision Point:** A `250` or `252` response typically indicates a valid user, while a `550` error indicates the user is unknown.

### Technique 2: POP3 Manual Interaction

Connect to port 110 to test the `USER` command.

**Workflow for POP3:**

1. Connect: `telnet <TARGET_IP> 110`.
2. Issue command: `USER <USERNAME>`.
3. **Decision Point:** An `+OK` response confirms the user exists; `-ERR` indicates they do not.

### Technique 3: Automated Enumeration

Automate the process for large wordlists using `smtp-user-enum`.

```
smtp-user-enum -M <MODE> -U <USER_LIST> -D <DOMAIN> -t <TARGET_IP>
```

- **Modes:** `VRFY`, `EXPN`, or `RCPT`.
- **Context:** Use when manual testing confirms the server responds to these commands.

### Technique 4: Cloud-Specific Enumeration (Office 365)

Cloud providers require specialized tools like `o365spray` to abuse custom features for enumeration.

1. **Validate Domain:**
    
    ```
    python3 o365spray.py --validate --domain <DOMAIN>
    ```
    
2. **Enumerate Users:**
    
    ```
    python3 o365spray.py --enum -U <USER_LIST> --domain <DOMAIN>
    ```
    

---

## Password Attacks

After gathering valid usernames, attempt to identify valid credentials.

### Protocol Brute Force (Custom Servers)

Use `Hydra` against standard protocols like POP3, IMAP, or SMTP.

```
hydra -L <USER_LIST> -p '<PASSWORD>' -f <TARGET_IP> <SERVICE>
```

- **Service:** `pop3`, `imap`, or `smtp`.
- **Context:** Effective against custom implementations but often blocked by cloud providers.

### Cloud Password Spraying

For Microsoft 365, use `o365spray` to bypass common blocks associated with standard tools.

```
python3 o365spray.py --spray -U <VALID_USERS_LIST> -p '<PASSWORD>' --count 1 --lockout 1 --domain <DOMAIN>
```

- **Strategy:** Use a single password across many users (`--count 1`) and respect lockout timers (`--lockout 1`) to avoid account lockouts.

---

## Protocol-Specific Attacks: Open Relay

An **open relay** is a misconfigured SMTP server that allows unauthenticated users to send emails to external domains.

### 1. Identification

Identify if the server allows re-routing of mail from any source.

```
nmap -p25 -Pn --script smtp-open-relay <TARGET_IP>
```

### 2. Exploitation (Phishing/Spoofing)

If identified, use the server to send spoofed emails, making them appear to originate from the trusted open relay server.

```
swaks --from <SPOOFED_SENDER> --to <TARGET_USER> --header 'Subject: <SUBJECT>' --body '<MESSAGE_CONTENT>' --server <TARGET_IP>
```

- **Attack Implication:** Masks the true source of the email, increasing the success rate of phishing campaigns.

---

## Dangerous Misconfigurations

|Setting|Risk|Attack Vector|
|:--|:--|:--|
|**Open Relay**|Unauthenticated mail forwarding.|Phishing and email spoofing.|
|**VRFY/EXPN Enabled**|Information disclosure.|Targeted username enumeration.|
|**Anonymous Authentication**|Unrestricted access to SMTP.|Unauthorized email sending/enumeration.|
|**Default POP3 Behavior**|Messages removed from server after download.|Potential data loss for the user; limited access across devices.|