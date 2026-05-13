## OpenSMTPD RCE (CVE-2020-7247)

OpenSMTPD <= 6.6.2 banner detected on port 25; requires no authentication for unauthenticated command execution.

Manual interaction to trigger shell command execution via sender field injection

```
nc -nv <TARGET_IP> 25
# Connect and compose email
# In the sender field, use: ;<COMMAND>
```

> ⚠️ Gap: Source lacks specific SMTP protocol verbs (e.g., HELO, MAIL FROM, RCPT TO) required to reach the sender field during manual interaction.

- Service running with **root privileges** to bind to port 25 allows the injected command to execute with elevated permissions.
    
- Command length is strictly limited to **64 characters**.
    
- **Exceeding 64 characters** will result in the command failing to execute.
    

## Inbox Enumeration and Brute Forcing

Authentication services like OWA, IMAP, or SMTP are exposed; requires brute-forcing credentials or direct IMAP interaction.

Manual IMAP interaction for inbox enumeration

```
nc -nv <TARGET_IP> <PORT>
```

- SMTP → Use for brute-forcing users to facilitate phishing.
- IMAP client → Use for enumerating a user's inbox once access is gained.

> ⚠️ Gap: Source identifies brute-forcing and IMAP enumeration as techniques but provides no specific command syntax or flags for tools.

- **Invalid credentials** will prevent access to the IMAP client or inbox enumeration.