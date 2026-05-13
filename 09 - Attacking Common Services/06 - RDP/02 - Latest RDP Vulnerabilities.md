
1. Identify open RDP service on `TCP/3389`
2. Verify target is a Windows variant pre-dating or missing the May 2019 security updates,
3. Confirm with client before proceeding due to high risk of **system instability** and **BSoD**
4. Send manipulated initialization request to trigger Use-After-Free (UAF) in virtual channel creation,
5. Overwrite freed kernel memory with instructions for CPU execution
6. Catch reverse shell initiated from `LocalSystem` account

---

## BlueKeep RDP Exploitation (CVE-2019-0708)

### When to use

Unauthenticated RDP access on `TCP/3389` where legacy software or costly maintenance has prevented security patching,.

### Commands

> ⚠️ Gap: The source material describes the technical flow and logic of the UAF exploit but does not provide specific CLI tools, exploit module names, or syntax for execution. Use of external tools is required to implement the following logical stages.

Trigger the vulnerability during the settings exchange phase before user authentication.

```
Source: Manipulated initialization request
Process: Create virtual channel
Privileges: LocalSystem
Destination: Redirect to kernel process
```

### Dangerous / misconfigured settings

- RDP service enabled without Network Level Authentication (NLA)
- Legacy Windows versions or unpatched systems missing May 2019 updates

### Gotchas

**Blue Screen of Death (BSoD)**. The exploit is known to cause immediate system crashes and instability; client authorization is mandatory.

## Remote Code Execution Stage

### When to use

Initial settings exchange successfully redirected to a kernel process to allow memory manipulation.

### Commands

Inject the secondary payload to free kernel memory and point the CPU to arbitrary instructions.

```
Source: Attacker payload inserted into kernel process
Process: Free kernel memory / CPU redirection
Privileges: LocalSystem
Destination: Reverse shell to <ATTACK_IP>
```

### Edge cases

- Large organizations (e.g., hospitals) are higher-probability targets due to reliance on specific legacy libraries that prevent patching.
- Approximately 25% of the 950,000 systems identified in 2019 remain vulnerable.

### Gotchas

**Restricted kernel memory**. Exploitation requires writing directly to kernel memory; failure to align instructions correctly will result in **total system failure** rather than a shell,.