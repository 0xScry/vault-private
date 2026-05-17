## Methodology

1. **Enumeration and identification** — locate promising exploits or security bypasses.
2. **Payload selection** — determine the method to deliver a remote shell session to the target.
3. **Shell execution** — trigger the exploit (e.g., EternalBlue) to obtain interactive CLI access.
4. **Post-exploitation** — leverage shell for file transfers, privilege escalation, and pivoting.
5. **Persistence** — establish a permanent foothold to maintain access over time.

---

## Interactive Shell Access

**When to use** Direct access to the OS, system commands, and file system is required after identifying a vulnerability. Use CLI over VNC/RDP when **stealth and speed** are prioritized, as graphical shells are **easier to detect**.

**Commands** The source lists shell environments but lacks specific execution strings for reverse/bind connections.

> ⚠️ Gap: Specific syntax for Bash, Zsh, cmd, and PowerShell shell invocation is not provided in the source text.

**Gotchas** **Limited access** — failing to establish a shell session restricts activity to basic enumeration and prevents escalation or pivoting.

## Web Shell Deployment

**When to use** A vulnerability exists allowing file or script uploads to the target. Minimum requirement is the ability to call the uploaded script via a browser to issue instructions.

**Commands** Control the web shell by calling the script within a browser window.

```
http://<TARGET_IP>/<FILE_PATH>
```

**Edge cases** Web shells are specifically used to interact with the underlying host via HTTP when standard OS shell ports may be blocked.

**Gotchas** **Destructive actions** — web shells can potentially perform irreversible damage to the underlying host if commands are not validated.

## Payload Delivery

**When to use** A vulnerability has been identified and a remote shell session must be established.

**Commands** Example scenario: triggering EternalBlue on a Windows host to gain a cmd prompt.

> ⚠️ Gap: Specific payload generation tools and exploit execution commands are not defined in the source.

**Dangerous / misconfigured settings**

- **Graphical shells (VNC/RDP)** — easier for defenders to notice and slower to navigate than CLI.
- **Manual repetition** — failing to automate actions on the CLI increases engagement time.