## Pivoting and Tunneling Operations

When needing to influence teammate actions or provide results for senior testers during joint assessments.

> ⚠️ Gap: Source material lacks specific command syntax, flags, and binary execution details for SSH and SOCKS implementation.

- **Defenders can spot pivots** by identifying traffic tunneled through non-standard routes.
- **Host compromise is detectable** if a host is used as a pivot point for lateral movement.

## SSH Tunneling and Multi-Protocol Proxies

When requiring encapsulated traffic across various protocols to bypass network restrictions.

> ⚠️ Gap: Source identifies SpecterOps and SANS as references for these tools but does not provide the command-line interface instructions for their use.

- **Tool selection** depends on whether the target environment requires standard SSH tunneling or proxying over alternative protocols.

## Active Directory Enumeration and Pivoting

When encountering an AD environment with multiple domains requiring chained pivoting skills.

Use a pivot to establish an initial entry point into the target subnet:

```
<PIVOT_IP>
```

1. Identify the entry point for the internal network, such as the Ascension lab target
2. Establish a tunnel to facilitate AD enumeration and attacking skills
3. Chain pivoting techniques to move between different AD domains

- **Pivoting through corporate networks** requires chaining techniques to reach isolated segments.
- **Skill level determines autonomy** in performing lateral movement and port forwarding during day-to-day duties.