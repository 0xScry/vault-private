## DNS Zone Transfer (AXFR)

Reconnaissance to bypass brute-forcing for subdomain discovery; requires a target DNS server with **unrestricted access controls**.

Request a full zone transfer from a specific name server

```
dig axfr @<TARGET_IP> <DOMAIN>
```

- Access controls allowing **any client** to initiate transfers.
    
- Outdated practices failing to restrict transfers to **trusted secondary servers**.
    
- Failed transfer attempts can still leak information regarding the server's **security posture** or internal configuration.
    

**Unauthorized transfer rejection** occurs when the server is properly hardened to only communicate zone data with legitimate secondary name servers.

> ⚠️ Gap: The technique requires identifying the authoritative name server for the domain first; running AXFR against a standard recursive resolver or the wrong authority will result in a silent failure or timeout.