## Dynamic Port Forwarding

Host is **locked down** and requires a pivot point without transferring external binaries; PuTTY or **Plink** must be present on the system or an accessible **file share**.

Start a **dynamic port forward** and listener on a specific local port over SSH:

```
plink -ssh -D <PORT> <USERNAME>@<PIVOT_IP>
```

- **Plink** -> `plink -ssh -D <PORT>` -> Prefer for **legacy Windows** (pre-Fall 2018) or when living off the land to avoid **detection**.
- **SSH (Native)** -> `ssh -D <PORT>` -> Prefer on modern Windows builds where the native client is available.

**Gotchas** **Silent failure** occurs on newer Windows builds if you default to **Plink** instead of using the native SSH client when available.

## Windows Application Tunneling

Windows-based attack host needs to route desktop client applications through an established **SOCKS proxy**.

1. Start the **Plink** session to open the local listener port:

```
plink -ssh -D <PORT> <USERNAME>@<PIVOT_IP>
```

2. Configure **Proxifier** Proxy Server settings to intercept local traffic:

```
Address: 127.0.0.1
Port: <PORT>
Protocol: SOCKS4
```

3. Launch the target application to operate through the **SOCKS tunnel**:

```
mstsc.exe
```

**Gotchas** **Connection failure** will result if the application is launched before the **Plink** listener is fully established and authenticated.