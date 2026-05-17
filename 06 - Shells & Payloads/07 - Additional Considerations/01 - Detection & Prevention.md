### METHODOLOGY

1. **Initial Access Phase**: Target public-facing web applications, misconfigured SMB/authentication protocols, or known vulnerabilities in bastion hosts to establish a foothold.
2. **Execution Phase**: Deploy payloads via web browser command injection, PowerShell one-liners using **PsExec**, or Metasploit exploits.
3. **Command & Control (C2) Phase**: Blend traffic with standard protocols (HTTP/S, DNS, NTP) or common applications (Slack, Discord, MS Teams) to avoid detection.
4. **Network Evasion**: Check for **Deep Packet Inspection (DPI)** or cloud-based L7 visibility. Use encrypted channels to prevent command reconstruction from cleartext traffic like **Netcat**.
5. **Host Persistence**: Monitor for active command-line logging and **Windows Defender** status before executing local administrative commands.

---

## Network Visibility and Traffic Analysis

**When to use** Detecting unencrypted callbacks or high-frequency NetFlow to suspicious ports like **4444** after payload execution.

**Commands** View cleartext TCP streams to reconstruct executed commands and directory listings

```
wireshark
```

**Tool comparison**

- Netbrain
    - `netbrain`
    - Prefer for interactive visual topologies combining documentation and remote management.
- Draw.io
    - `draw.io`
    - Prefer for static manual diagramming of network traffic flow.

**Dangerous / misconfigured settings**

- Unencrypted cleartext channels (Netcat) for bind or reverse shells.
- Standard ports (80, 443) used for non-standard, unencrypted traffic.

**Gotchas** **Deep Packet Inspection** can act as network-level anti-virus and block payloads even if host-level execution succeeds.

## Host-Level Execution and Persistence

**When to use** Establishing long-term access after gaining an initial foothold through code execution.

**Commands** Create a new user for persistence

```
net user <USERNAME> <PASSWORD> /add
```

Elevate user privileges to local administrator

```
net localgroup <SERVICE_NAME> <USERNAME> /add
```

**Dangerous / misconfigured settings**

- Disabled **Windows Defender** or firewall profiles (Domain, Private, Public).
- Lack of **command-line logging** for auditing administrative actions.
- Missing patch management for critical OS updates.

**Edge cases**

- **Server-side AV** might be disabled or have exceptions due to performance overhead, providing a window for shell establishment.

**Gotchas** **Command-line logging** paired with NetFlow data allows defenders to quickly triage malicious user creation regardless of the account name used.

## Defense-in-Depth and Hardening

**When to use** Identifying potential roadblocks or mitigation strategies that prevent exploit success.

**Tool comparison**

- Windows Defender
    - `Windows Security`
    - Prefer for native endpoint protection and integrated firewall management.
- L7 Cloud Controllers (Cisco Meraki, Palo Alto)
    - `<VENDOR_PORTAL>`
    - Prefer for baseline traffic monitoring and detecting protocol deviations at the network edge.

**Dangerous / misconfigured settings**

- Over-privileged service accounts violating the **Principle of Least Privilege**.
- Flat network architecture without **Network Segmentation**.
- Allowing unapproved applications through the firewall without a change management process.

**Gotchas** **Application Whitelisting** can prevent the execution of unauthorized shell scripts and payloads even if they are successfully delivered to the host.

> ⚠️ Gap: Source mentions "PowerShell one-liner via PsExec" but lacks the specific syntax for the bypass or execution flags required to avoid immediate AV/EDR triggers.

> ⚠️ Gap: Source notes that DPI can act as network AV but does not specify which encryption protocols (SSL/TLS) are required to bypass specific inspection signatures.