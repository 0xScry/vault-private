## SOCKS Tunneling via RDP

Windows-only environment where SSH is restricted but RDP is available between the foothold and internal targets.

1. Register the **Dynamic Virtual Channel (DVC)** plugin on the Windows foothold to enable tunneling over the Remote Desktop Service.
2. Initiate an RDP connection from the foothold to the target host to trigger the plugin listener on `127.0.0.1:1080`.
3. Execute the server-side component on the target host using **Admin privileges** to bridge the tunnel.
4. Verify the local SOCKS listener is active and configure a proxy tool to route traffic through the established DVC.

---

Register the DLL on the foothold to hook into the Remote Desktop Service

```
regsvr32.exe SocksOverRDP-Plugin.dll
```

Check if the plugin successfully opened the SOCKS listener on the foothold

```
netstat -antb | findstr 1080
```

Start the server component on the target host to finalize the tunnel

```
SocksOverRDP-Server.exe
```

**Dangerous / misconfigured settings**

- Running the server component without **Admin privileges**.
- Failing to register the DLL on the foothold before initiating the RDP session.

**Edge cases**

- **Slow performance** or high latency during multi-layered RDP sessions requires setting the RDP Experience tab to **Modem (56 kbps)**.

**Gotchas**

- **Admin privileges** are mandatory on the target host for `SocksOverRDP-Server.exe` to function.
- The SOCKS listener only binds to **127.0.0.1:1080** on the foothold.