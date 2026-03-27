# SSH Tunneling with Plink.exe and Proxifier

### Technique Overview

**Plink (PuTTY Link)** is a Windows command-line SSH tool used to create **dynamic port forwards** and **SOCKS proxies**.

**When to use:**

- When operating from a **Windows-based attack host**.
- When **living off the land** on a compromised Windows pivot host that lacks a native SSH client (common in pre-2018 versions) but has PuTTY/Plink installed.
- To **avoid detection** by using existing, legitimate administrative tools.

**Attack Implications:**

- Unlocks the ability to route traffic from Windows desktop applications (like RDP) through a pivot point into an internal network.

---

### Command Reference: Plink.exe

|Parameter|Description|
|:--|:--|
|`-ssh`|Specifies the use of the SSH protocol.|
|`-D <PORT>`|Initiates a **dynamic port forward** on the specified local port.|

---

### Operational Workflow

#### 1. Establish the Dynamic Port Forward

Use this command to create an SSH session between your Windows host and a pivot host, opening a local SOCKS listener.

```
plink -ssh -D <PORT> <USERNAME>@<PIVOT_IP>
```

- **Goal:** Starts an SSH session and begins listening on the specified `<PORT>` (e.g., 9050) for incoming traffic to be proxied.

#### 2. Tunneling Traffic with Proxifier

Since many Windows applications do not natively support SOCKS proxies, **Proxifier** is used to force application traffic through the Plink tunnel.

1. **Configure Proxy Server:** Add a new proxy entry in Proxifier settings.
    - **Address:** `127.0.0.1`
    - **Port:** The `<PORT>` defined in the Plink command.
    - **Protocol:** **SOCKS4**.
2. **Launch Target Application:** Once the profile is active, applications will operate through the proxy chain.
3. **Example (RDP to Internal Target):**
    
    ```
    mstsc.exe
    ```
    
    - **Goal:** Initiate an RDP session with a `<TARGET_IP>` that is only reachable through the established tunnel.

---

### Scenario Context

- **Inbound Restrictions:** If a pivot host is moderately locked down and you cannot upload custom tools, check for `plink.exe` in common locations or file shares to establish a pivot point.