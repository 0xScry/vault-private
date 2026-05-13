## Methodology

1. Review external data footprint including job postings and DNS records to identify user naming context and internal tooling
2. Monitor network traffic for bursts or high-volume packets from single sources indicating active scanning
3. Enforce **SMB signing** and strong encryption to break relay and Man-in-the-Middle techniques
4. Audit for **RC4 encryption** in Kerberos mechanisms and replace with stronger schemes or **Group Managed Service Accounts** to neutralize Kerberoasting
5. Implement **AppLocker** policies to restrict command shell access and unauthorized application execution
6. Populate the **Protected Users** group to prevent credential caching in memory, starting with staged testing to avoid availability issues

---

## Enumerating Protected Users

Mapping high-value targets protected against memory-based credential abuse and identifying accounts that will not yield cleartext or hashes from host memory

Check group membership and description to identify restricted accounts

```
Get-ADGroup -Identity "Protected Users" -Properties Name,Description,Members
```

- **Dangerous / misconfigured settings**
    
    - Adding all privileged users to the group simultaneously without staged testing
- **Gotchas**
    
    - **Account lockout** can occur unexpectedly due to authentication failures triggered by group restrictions

## Mitigating Poisoning and Relay

Neutralizing Man-in-the-Middle and traffic spoofing by validating communication endpoints

Enable SMB signing to verify sender and recipient identity via hashed authentication codes

```
Match/Replace: Set SMB Message Signing to Required
```

> ⚠️ Gap: Source mentions SMB signing as a defense but does not provide the specific registry or GPO paths to enable it.

- **Dangerous / misconfigured settings**
    
    - Disabled or optional **SMB message signing**
    - Weak encryption mechanisms for network traffic
- **Gotchas**
    
    - **Relay attacks break** immediately once signing is enforced as the attacker cannot spoof the hashed authentication

## Preventing Kerberoasting

Identifying and hardening service accounts against ticket extraction and offline cracking

Audit account permissions to identify excessive group memberships and service principal names

```
Action: Audit excessive group membership for service accounts
```

- **Dangerous / misconfigured settings**
    
    - Utilizing **RC4 encryption** for Kerberos authentication
    - Weak password policies for service accounts
- **Edge cases**
    
    - Migration to **Group Managed Service Accounts (gMSA)** makes Kerberoasting technically impossible

## Defending Internal Reconnaissance

Obscuring the internal environment and detecting active discovery attempts through traffic analysis

Configure host-based protections to drop discovery traffic

```
Action: Set Windows Firewall/EDR to not respond to ICMP
```

- **Dangerous / misconfigured settings**
    
    - Firewall/EDR configured to respond to **ICMP traffic**
    - Lack of centralized logging for **Event IDs 4624 and 4648**
- **Gotchas**
    
    - **Alerting triggers** when suspicious bursts of packets originate from a single source host