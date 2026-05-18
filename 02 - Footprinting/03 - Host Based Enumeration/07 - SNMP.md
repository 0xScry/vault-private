1. Identify UDP 161 via scanning.
2. Attempt enumeration using the **default community string** `public`.
3. If default access fails or is restricted, brute-force community strings using `onesixtyone` and `SecLists`.
4. Once a valid string is identified, use `snmpwalk` to dump the **Management Information Base** tree.
5. For high-speed or bulk **Object Identifier** enumeration across specific ranges, use `braa`.
6. Inspect output for **plain text** credentials, software versions, and network configuration.

---

## Community String Discovery

UDP 161 is open and the **community string** is unknown or default strings fail to return data.

Brute-force community strings using a wordlist to identify valid access credentials.

```
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <TARGET_IP>
```

> ⚠️ Gap: If the community string is bound to a specific source IP, brute-forcing will **fail silently** without a spoofed source or pivot from an authorized host.

- **v1/v2c community strings** are transmitted in **plain text** and can be intercepted via packet capture.
- Administrators often use the hostname or predictable patterns for community names in large environments.

## SNMP Enumeration

A valid community string is known and full system information is required.

Query the OID tree to extract system descriptions, mount points, and installed software.

```
snmpwalk -v2c -c <PASSWORD> <TARGET_IP>
```

Query specific OID ranges rapidly when `snmpwalk` is too slow or limited.

```
braa <PASSWORD>@<TARGET_IP>:.1.3.6.*
```

- `snmpwalk`
    
    - `snmpwalk -v2c -c <PASSWORD> <TARGET_IP>`
    - Prefer for standard, human-readable enumeration of the full **MIB**.
- `braa`
    
    - `braa <PASSWORD>@<TARGET_IP>:<OID_PATTERN>`
    - Prefer for brute-forcing individual OIDs or mass-querying specific branches.
- `rwuser noauth`: Provides full **OID** tree access without any authentication.
    
- `rwcommunity <PASSWORD> <TARGET_IP>`: Provides read-write access to the full tree from any source.
    
- **Lack of encryption** in v1 and v2c allows all transmitted data and community strings to be read by anyone with network access.
    
- **SNMPv3 complexity** often results in organizations reverting to v2c, leaving credentials exposed.
    

## Trap Monitoring

Monitoring for asynchronous alerts sent from a server to a client.

Listen for unsolicited data packets triggered by specific server-side events on UDP 162.

```
# No command provided in source for trap listening
```

- **Traps** are sent to the client without an explicit request when specific events occur.
- Requires the device to be configured with the client's destination IP. [i] [ii] [iii] [iv] [v] [vi] [vii] [viii] [ix] [x] [xi] [xii] [xiii] [xiv]