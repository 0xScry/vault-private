## Automatic Request Modification

When to use: Filter-blocked **User-Agents** or recurring header/body strings requiring persistent global replacement.

Navigate to Burp replacement rules

```
Proxy > Proxy settings > HTTP match and replace rules
```

Regex rule to swap User-Agent (Burp)

```
Type: Request header
Match: ^User-Agent.*$
Replace: User-Agent: HackTheBox Agent 1.0
Regex match: True
```

Open ZAP Replacer

```
CTRL+R
```

Global User-Agent replacement rule (ZAP)

```
Rule: HTB User-Agent
Type: Request Header String
Match: User-Agent
Replace: HackTheBox Agent 1.0
Regex: True
```

### Tool comparison

- Burp Suite
    - `Proxy > Proxy settings > HTTP match and replace rules`
    - Prefer for **Regex match** on specific header lines
- ZAP
    - `Options > Replacer`
    - Prefer when limiting rules to specific **Initiators**

### Dangerous / misconfigured settings

- ZAP default **Initiators** set to "Apply to all HTTP(S) messages" triggers modifications across all proxied traffic globally.

### Gotchas

- **Regex match: False** causes rule failure if the match field contains a pattern instead of an exact literal string.

## Automatic Response Modification

When to use: Client-side validation like restricted input types or field lengths that revert on refresh and block payload delivery.

Change input type to allow non-numeric injection (Burp)

```
Type: Response body
Match: type="number"
Replace: type="text"
Regex match: False
```

Extend field character limits (Burp)

```
Type: Response body
Match: maxlength="3"
Replace: maxlength="100"
Regex match: False
```

### Gotchas

- **Manual interception** modifications are temporary; automated rules are required for changes to persist across page refreshes.