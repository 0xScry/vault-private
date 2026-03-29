# IMAP & POP3 Enumeration and Interaction

## Protocol Overview

**IMAP (Internet Message Access Protocol)** and **POP3 (Post Office Protocol)** are client-server protocols used to access emails on a remote server.

- **IMAP (Port 143/993):** Designed for **online management**. It supports synchronization across multiple clients, folder structures, and server-side browsing.
- **POP3 (Port 110/995):** Designed for simple **retrieval**. It offers limited functionality: listing, retrieving, and deleting emails.

**Attack Implication:** Both protocols transmit credentials and data in **plain text** unless encrypted via SSL/TLS (typically ports 993/995). If unencrypted, an attacker can intercept usernames and passwords.

---

## Service Footprinting

The goal of footprinting is to identify the service version, available capabilities, and potential domain information embedded in SSL certificates.

### 1. Initial Scan

Use Nmap to identify open ports and service versions. The default ports are **110/995 (POP3)** and **143/993 (IMAP)**.

```
sudo nmap <TARGET_IP> -sV -p110,143,993,995 -sC
```

- **Why it matters:** The output reveals **capabilities** (available commands) and **SSL certificate** details (Common Name, Organization), which often provide internal domain names like `<DOMAIN>`.

### 2. Interaction via cURL

If you possess credentials, use `curl` to quickly list mailboxes.

```
curl -k 'imaps://<TARGET_IP>' --user <USERNAME>:<PASSWORD>
```

- **Scenario:** Use the `-v` (verbose) flag to view the **TLS version**, certificate details, and the **service banner**, which may contain the specific mail server version.

---

## Direct Interaction & Authentication

After discovering the service, you can interact directly via text-based commands.

### Encrypted Interaction

For services running on ports 993 or 995, use OpenSSL to establish an encrypted session.

**POP3 Encrypted:**

```
openssl s_client -connect <TARGET_IP>:995
```

**IMAP Encrypted:**

```
openssl s_client -connect <TARGET_IP>:993
```

### Protocol Command Reference

#### IMAP Commands (Port 143/993)

Used for managing mailboxes and reading messages.

|Command|Description|
|:--|:--|
|`1 LOGIN <USERNAME> <PASSWORD>`|Authenticates the user.|
|`1 LIST "" *`|Lists all directories/folders.|
|`1 CREATE "INBOX"`|Creates a new mailbox.|
|`1 DELETE "INBOX"`|Deletes a mailbox.|
|`1 RENAME "Old" "New"`|Renames a mailbox.|
|`1 SELECT INBOX`|Selects a mailbox to access messages.|
|`1 FETCH <ID> all`|Retrieves data for a specific message.|
|`1 LOGOUT`|Closes the connection.|

#### POP3 Commands (Port 110/995)

Used for basic email retrieval.

|Command|Description|
|:--|:--|
|`USER <USERNAME>`|Identifies the user.|
|`PASS <PASSWORD>`|Authenticates the user.|
|`STAT`|Requests the number and total size of saved emails.|
|`LIST`|Requests the number and size of all individual emails.|
|`RETR <ID>`|Retrieves a specific email by ID.|
|`DELE <ID>`|Deletes a specific email by ID.|
|`CAPA`|Lists server capabilities.|
|`QUIT`|Closes the connection.|

---

## Misconfigured & Dangerous Settings

Administrators may enable debugging or verbose logging, which can leak sensitive data into log files.

|Setting|Attack Implication|
|:--|:--|
|`auth_debug`|Enables all authentication debug logging; may reveal internal logic.|
|`auth_debug_passwords`|**High Risk:** Logs the actual submitted passwords and their schemes.|
|`auth_verbose`|Logs unsuccessful login attempts and the reasons for failure.|
|`auth_verbose_passwords`|Logs passwords used for authentication (may be truncated).|
|`auth_anonymous_username`|Specifies a username for anonymous logins, potentially allowing access without a known account.|

---

## Operational Workflow: Credential Testing

If credentials (e.g., `<USERNAME>:<PASSWORD>`) are recovered from other services like SMTP, they should be tested against IMAP and POP3 to gain access to mailbox content.

1. **Establish Connection:** Use `openssl` or `nc`.
2. **Authenticate:** Use the `LOGIN` (IMAP) or `USER/PASS` (POP3) commands.
3. **Enumerate Folders:** Use `LIST` to find sensitive locations like "Important" or "Sent".
4. **Read Mail:** Use `FETCH` or `RETR` to extract message contents for further reconnaissance.