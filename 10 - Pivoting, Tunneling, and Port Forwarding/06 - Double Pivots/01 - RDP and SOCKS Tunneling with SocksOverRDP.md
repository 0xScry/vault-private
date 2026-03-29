# RDP and SOCKS Tunneling with SocksOverRDP

### Scenario and Purpose

**Use this technique when limited to a Windows network environment where SSH pivoting is unavailable**. This method leverages **Dynamic Virtual Channels (DVC)** within the Remote Desktop Service—typically used for clipboard or audio sharing—to tunnel arbitrary network packets.

### Tool Requirements

|Tool|Location|Purpose|
|:--|:--|:--|
|`SocksOverRDP-Plugin.dll`|**Pivot Host**|Registered as a DVC plugin to enable tunneling.|
|`SocksOverRDP-Server.exe`|**Target Host**|Handles the server-side component of the tunnel; requires Admin privileges.|
|`Proxifier`|**Pivot Host**|Acts as a proxy server to route traffic through the SOCKS listener.|

---

### Operational Workflow

#### 1. Plugin Registration (Pivot Host)

After transferring the `SocksOverRDP` binaries to the compromised pivot host, the plugin must be registered to the system to interact with the RDP service.

```
regsvr32.exe SocksOverRDP-Plugin.dll
```

- **Goal:** Load the DLL into the Windows registry so that subsequent RDP connections recognize the plugin.

#### 2. Establish RDP Connection

Connect from the pivot host to the internal target machine. Upon connection, the registered plugin will automatically start a SOCKS listener on the pivot host's loopback address.

```
mstsc.exe /v:<TARGET_IP>
```

- **Decision Point:** Ensure a prompt appears confirming the **SocksOverRDP plugin is enabled** and listening on `127.0.0.1:1080`.

#### 3. Tunnel Initialization (Target Host)

The server-side component must be executed on the target machine to complete the circuit.

1. Transfer `SocksOverRDP-Server.exe` to the `<TARGET_IP>`.
2. **Execute with Admin privileges** on the target.

#### 4. Verification and Routing (Pivot Host)

Confirm the SOCKS listener is active on the pivot host before routing traffic.

```
netstat -antb | findstr 1080
```

- **Goal:** Verify the pivot host is **LISTENING** on port `1080`.

#### 5. Proxy Traffic with Proxifier

To use the tunnel for other tools or additional RDP sessions, configure **Proxifier** on the pivot host to forward all packets to `127.0.0.1:1080`.

- **Result:** Traffic is routed through the pivot host, tunneled over RDP via `SocksOverRDP-Server.exe`, and can reach deeper internal targets (e.g., reaching a second target at `<TARGET_IP_2>`).

---

### Optimization and Edge Cases

#### RDP Performance Considerations

Tunnelling multiple sessions can cause significant lag.

- **Optimization:** In `mstsc.exe`, navigate to the **Experience** tab and set the performance to **Modem (56 kbps)** to improve responsiveness during the engagement.
- - **Note:** Windows Defender may delete the binary immediately on download. If the file disappears after transfer, disable real-time monitoring (`Set-MpPreference -DisableRealtimeMonitoring $true`) or add an exclusion path (`Add-MpPreference -ExclusionPath "C:\path"`). If neither works, try transferring to `C:\Windows\Temp`.