## Domain Trust Methodology

1. Identify established trust relationships to find **transitive** or **bidirectional** paths into higher-value domains from a compromised subdomain.
2. Check `IntraForest` and `ForestTransitive` properties to determine if you are moving within a forest or across a forest boundary.
3. Verify `TrustDirection`; **authentication failure** on a one-way trust will kill any cross-domain enumeration or exploitation.
4. Map the full trust topology to identify **cross-link** or **external** trusts that bypass standard hierarchy.
5. Execute cross-domain enumeration for users and high-value targets in the trusted domain to find **administrative access** or **Kerberoasting** opportunities.

---

## Domain Trust Enumeration

Found a foothold and need to identify lateral movement paths to other domains or the forest root.

### Built-in Windows Tools

Identify trusts using native binaries or the Active Directory PowerShell module when avoiding non-native tools.

Enumerate all trusts for the current domain using the AD PowerShell module

```
Get-ADTrust -Filter *
```

Query the domain for basic trust listing and directionality

```
netdom query /domain:<DOMAIN> trust
```

### Offensive Toolsets

Map complex trust relationships and visualize the authentication flow.

List all trusts for the current domain including attributes and direction

```
Get-DomainTrust
```

Perform a detailed mapping of all trusts discovered in the environment

```
Get-DomainTrustMapping
```

- Tool comparison
    - `Get-ADTrust` -> `Get-ADTrust -Filter *` -> use when the **Active Directory PowerShell module** is already present on the target.
    - `netdom` -> `netdom query /domain:<DOMAIN> trust` -> use when restricted to **built-in command-line tools**.
    - `PowerView` -> `Get-DomainTrust` -> prefer for detailed **TrustAttributes** and mapping data during active pivoting.
    - `BloodHound` -> **Map Domain Trusts** query -> use for **visualizing** complex bidirectional paths across the forest.

**Gotchas** **Authentication failure** occurs if the trust direction does not allow your current security principal to traverse the boundary.

## Cross-Trust Resource Discovery

Trust is confirmed and directionality allows for remote enumeration of the target domain.

### Resource Enumeration

List all user accounts in a child or trusted domain to find potential targets for further attacks.

```
Get-DomainUser -Domain <DOMAIN> | select SamAccountName
```

Identify domain controllers in the trusted domain for targeting and mapping.

```
netdom query /domain:<DOMAIN> dc
```

Enumerate workstations and servers in the trusted domain to identify high-value targets like SQL servers.

```
netdom query /domain:<DOMAIN> workstation
```

### Dangerous / misconfigured settings

- **Bidirectional** trusts allow users to authenticate in both directions, potentially exposing the principal domain to a compromised child.
- **Transitive** trusts automatically extend trust to every other domain in the forest, expanding the attack surface.
- **SelectiveAuthentication** set to **False** may allow broader access than intended across the trust.

**Gotchas** **In-scope verification** is required before targeting any discovered trusts to avoid violating the **Rules of Engagement**.