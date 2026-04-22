# Domain Trust Enumeration and Abuse

### Domain Trusts Overview

Domain trusts allow for **forest-forest** or **domain-domain (intra-domain) authentication**, enabling users to access resources or perform administrative tasks outside their home domain. Trusts are often established during mergers and acquisitions (M&A) to integrate new business units without migrating all objects immediately.

**Why It Matters:** Trusts are frequently set up incorrectly or left unreviewed, creating **unintended attack paths**. A vulnerability in a "softer target" (like a child domain or an acquired company) can provide an **"end-around" attack** route into the principal domain, such as using **Kerberoasting** against a trusted domain to obtain a user with administrative rights in the primary domain.

---

### Trust Characteristics

|Property|Description|Attack Implication|
|:--|:--|:--|
|**Transitive**|The trust is shared with others in the forest; if A trusts B and B trusts C, then A trusts C.|Extends the attack surface across multiple levels of the forest.|
|**Non-Transitive**|A direct trust that is not extended to next-level child domains.|Limits movement to the specific established link.|
|**One-Way**|Authentication flows in only one direction.|Access is restricted to one-way resource sharing.|
|**Bidirectional**|Users in both domains can authenticate across the trust.|**Critical:** Allows for enumeration and attacks in both directions.|

---

### Enumeration Methodology

#### 1. Built-in PowerShell (AD Module)

Use this when restricted to built-in tools on a compromised host.

- **Goal:** Identify trust direction and transitivity settings for the current domain.

```
Import-Module activedirectory
Get-ADTrust -Filter *
```

- **Key Properties to Inspect:**
    - `IntraForest`: If `True`, it is likely a child domain trust.
    - `ForestTransitive`: If `True`, it is a forest or external trust.
    - `Direction`: Must be `BiDirectional` or appropriate for your target to allow authentication/enumeration.

#### 2. PowerView

PowerView provides a more detailed mapping of relationships and the ability to query across those trusts.

- **Goal:** Map trust types (parent/child, external, forest) and direction.

```
# Enumerate basic trust information
Get-DomainTrust

# Perform full domain trust mapping
Get-DomainTrustMapping
```

- **Goal:** Execute enumeration (e.g., user listing) in the discovered trusted domain.

```
Get-DomainUser -Domain <TARGET_DOMAIN> | select SamAccountName
```

#### 3. Netdom

A built-in Windows command-line tool for querying domain information.

- **Goal:** Retrieve a list of trusts, domain controllers, and workstations.

|Command|Purpose|
|:--|:--|
|`netdom query /domain:<DOMAIN> trust`|Lists all established trusts and their directions.|
|`netdom query /domain:<DOMAIN> dc`|Lists domain controllers in the target domain.|
|`netdom query /domain:<DOMAIN> workstation`|Lists workstations and servers in the target domain.|

#### 4. BloodHound

- **Goal:** Visualize complex trust relationships.
- **Technique:** Use the **Map Domain Trusts** pre-built query to identify bidirectional links and potential escalation paths.

---

### Operational Considerations

- **Authentication Requirement:** If you cannot authenticate across a trust (e.g., due to a one-way trust or restrictive settings), you cannot perform enumeration or attacks against that target.
- **Rules of Engagement (RoE):** Always verify if discovered trusts are in-scope. Enumerating or attacking a trusted domain belonging to a different business unit or partner (like an MSP) may be outside the permitted scope.