## Methodology

1. Scan for open mail ports (110, 143, 993, 995) to identify service types and extract **organization metadata** from SSL certificates.
2. Probe for **cleartext credentials** if encryption is not enforced on ports 110/143.
3. Use `curl` or `openssl` to grab banners and check for specific service versions or **anonymous login** capabilities.
4. If credentials exist, prioritize IMAP for directory navigation or POP3 for simple mail retrieval.

---

## Service Footprinting

Identify mail services, extract certificate info, and determine if **encryption** is mandatory.

Check ports for service versions and grab certificate details like common names or organization info.

```
sudo nmap <TARGET_IP> -sV -p110,143,993,995 -sC
```

Extract the service banner and TLS version to identify the specific mail server software.

```
curl -k 'imaps://<TARGET_IP>' --user <USERNAME>:<PASSWORD> -v
```

- Tool comparison
    
    - `nmap` -> `sudo nmap <TARGET_IP> -p110,143,993,995 -sC` -> prefer for initial discovery and **script-based** capability enumeration.
    - `curl` -> `curl -k 'imaps://<TARGET_IP>'` -> prefer for quick banner grabbing and testing **credential validaton**.
- Dangerous / misconfigured settings
    
    - `auth_debug`: Enables verbose authentication logging.
    - `auth_debug_passwords`: Logs actual submitted passwords.
    - `auth_anonymous_username`: Allows **anonymous access** via SASL.
    - Plaintext transmission: Services running without SSL/TLS on ports 110/143.
- Gotchas
    
    - **Self-signed certificates** will cause connection failures unless using `-k` in `curl` or ignoring verify errors in `openssl`.

## Authenticated IMAP Interaction

Access IMAP when **hierarchical folder management** or online mail manipulation is required.

Connect to IMAPS for manual command execution.

```
openssl s_client -connect <TARGET_IP>:imaps
```

Authenticate to the server; requires a leading **identifier** for every command.

```
1 LOGIN <USERNAME> <PASSWORD>
```

List all available directories and mailboxes.

```
1 LIST "" *
```

Select a specific mailbox to enable message access.

```
1 SELECT INBOX
```

Fetch all data for a specific message by its ID.

```
1 FETCH <ID> all
```

- Edge cases
    
    - Use `1 LSUB "" *` if you only need to see mailboxes the user has specifically **subscribed** to.
- Gotchas
    
    - **Missing identifiers** (e.g., the "1" before LOGIN) will cause the server to ignore or reject commands.
    
    > ⚠️ Gap: The source does not specify how to generate the Base64 hash required for `AUTHENTICATE PLAIN` seen in the output, which will cause manual authentication attempts via that specific command to fail. Use `1 LOGIN` for plaintext-equivalent manual auth.
    

## Authenticated POP3 Interaction

Access POP3 for basic **listing and retrieval** when IMAP is unavailable or unnecessary.

Connect to POP3S to interact with the encrypted service.

```
openssl s_client -connect <TARGET_IP>:pop3s
```

Authenticate via standard POP3 login flow.

```
USER <USERNAME>
PASS <PASSWORD>
```

Check the number of messages and total mailbox size.

```
STAT
```

Retrieve the full content of a specific email.

```
RETR <ID>
```

- Tool comparison
    
    - `IMAP` -> `1 SELECT INBOX` -> prefer for complex **folder structures**.
    - `POP3` -> `RETR <ID>` -> prefer for simple, linear **message recovery**.
- Gotchas
    
    - **DELE** commands in POP3 are often final; messages are removed from the server once the session closes if the delete flag is set.