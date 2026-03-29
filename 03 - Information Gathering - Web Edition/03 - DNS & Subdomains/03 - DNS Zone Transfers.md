### DNS Zone Transfers (AXFR)

**DNS Zone Transfers** are a reconnaissance technique used to uncover a target's entire DNS infrastructure. While intended for replicating records between primary and secondary name servers for consistency, **misconfigured access controls** allow unauthorized parties to download a complete zone file.

#### When to Use

- Use during the **reconnaissance phase** to map subdomains and internal IP addresses.
- Attempt this before or alongside brute-forcing, as it is less invasive and more efficient if successful.
- Even if the transfer is denied, the attempt can provide insights into the **DNS server's security posture**.

#### Operational Workflow

1. **Identify** the target's name server and the domain to be queried.
2. **Execute** a zone transfer request using the `dig` tool to ask for the full zone file (`axfr`).
3. **Analyze** the output for a comprehensive map of the target's environment, including subdomains, mail servers, and sensitive service records.

#### Command Reference

|Command|Description|Parameter|
|:--|:--|:--|
|`dig axfr @<DNS_SERVER> <DOMAIN>`|Requests a full zone transfer from a specific name server for the target domain.|`axfr`: Specifies the request for a full zone transfer.|

#### Attack Implications

A successful zone transfer reveals a wealth of sensitive data that unlocks further attack vectors:

- **A Records:** Reveals subdomains and their associated IP addresses.
- **MX Records:** Identifies mail servers.
- **TXT Records:** May contain site verification codes or sensitive configuration strings.
- **SRV Records:** Locates specific services (e.g., SIP) and their ports.
- **SOA/NS Records:** Provides details on the DNS infrastructure and administrative contacts.

#### Dangerous Misconfigurations

|Setting|Security Impact|
|:--|:--|
|**Unrestricted Access Controls**|Allows any client to initiate a zone transfer, leading to a full information leak of the zone file.|
|**Legacy/Human Error**|Outdated practices or manual errors can bypass modern defaults that restrict transfers to **trusted secondary servers**.|

#### Remediation

Modern DNS servers should be configured to allow zone transfers **only to trusted secondary servers** to ensure sensitive zone data remains confidential.