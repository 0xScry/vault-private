1. Evaluate if **Application Control Policy** blocks PowerShell execution.
2. Check for **web content filtering** preventing access to external repos like GitHub or cloud storage.
3. Test for **network firewall** egress blocks on Port 21.
4. Verify if **TCP Port 445** is open for outbound SMB connections.

---

## PowerShell Transfer

When script execution is required for enumeration but may be restricted by **Application Control Policy**.

> ⚠️ Gap: Source does not provide PowerShell transfer syntax.

- **Application Control Policy** will prevent the script from running or transferring.

## Certutil Transfer

When PowerShell is restricted and the environment allows outbound web traffic to external sites.

> ⚠️ Gap: Source does not provide Certutil command flags.

- **Web content filtering** frequently blocks GitHub, Dropbox, and Google Drive.

## FTP Transfer

When web-based transfer methods are filtered and the firewall allows Port 21 egress.

> ⚠️ Gap: Source does not provide Windows FTP client commands.

- **Network firewalls** commonly block outbound traffic on Port 21.

## SMB Transfer

When other outbound ports are restricted but **TCP Port 445** is open to the attack redirector.

Tool for setting up the listener on `<ATTACK_IP>`

```
impacket-smbserver
```

- **AV/EDR** or **host controls** may block the transfer of known malicious binaries or specific SMB operations.
- **IDS/IPS** systems can monitor and alert on uncommon SMB operations between internal hosts and external IPs.