## SMBGhost RCE (CVE-2020-0796)

Target is Windows 10 version 1903 or 1909 with SMB v3.1.1 compression active and **TCP/445** reachable.

> ⚠️ Gap: The source defines the integer overflow concept but lacks specific exploit script names or command-line syntax (e.g., Metasploit modules or Python PoCs) required to trigger the overflow and gain a shell.

### Attack Logic

1. Initiate SMB session negotiation to confirm **SMB v3.1.1** support
2. Send a malformed compressed message following the Negotiate Protocol Responses
3. Use an excessive data size that exceeds the integer variable limits of the SMB driver
4. Trigger an **integer overflow** in the memory-handling function to overwrite CPU instructions
5. Replace overwritten buffer space with attacker-controlled instructions for remote execution

### Dangerous / misconfigured settings

- SMB v3.1.1 compression enabled
- Lack of bounds checks in the SMB driver for session negotiation data
- SMB requests allowed over **TCP/445** from unauthenticated sources

### Gotchas

**Lack of compression support** during the protocol negotiation phase prevents the malformed packets from being processed by the vulnerable driver function. **Negative number wrap-around** occurs when a variable operation results in a negative number that the CPU returns as a positive integer, causing memory allocation mismatches.