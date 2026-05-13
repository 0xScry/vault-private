## PIVOTING METHODOLOGY

1. **Interface Enumeration**: Check for multiple **NICs** to identify **dual-homed** hosts connected to separate network segments.
2. **Routing Analysis**: Examine the host's routing table to identify the **default gateway** and specific routes to internal subnets.
3. **Port/Service Audit**: Identify listening services on open ports to find entry points or internal pivot targets.
4. **Technique Selection**:
    - **Lateral Movement**: Spread wide within the same segment to escalate privileges or access shared credentials.
    - **Pivoting**: Move deeper into previously unreachable segments by crossing network boundaries.
    - **Tunneling**: Encapsulate traffic within protocols like **HTTP/HTTPS** or **SSH** to obfuscate **Command & Control** and bypass detection.

---

## Interface Enumeration

When to use: Initial landing on a host to identify **private IP addresses** or **tunnel interfaces** that indicate access to internal lab networks.

Display interface configurations and assigned IPs on Linux or macOS

```
ifconfig
```

Display interface configurations, subnet masks, and gateways on Windows

```
ipconfig
```

- **Gotchas**: **VPN/Tunnel** connectivity is identified by a `tun0` interface; lab targets are unreachable if this interface is down.

## Routing Table Analysis

When to use: Determining how a compromised host forwards traffic and identifying reachable internal subnets not present on local interfaces.

View the kernel IP routing table for gateways and interface mappings

```
netstat -r
```

Alternative Linux command for displaying routing decisions and directly connected networks

```
ip route
```

- **Edge cases**: Hosts in a **dual-stack** configuration may reach resources via **IPv6** even if IPv4 routes are restricted.
- **Gotchas**: **Gateway of last resort** (default route) will attempt to handle any traffic destined for subnets not explicitly defined in the table.

## Pivot Implementation

When to use: Bridging network boundaries through a compromised host to access isolated segments like **DMZs** or operational environments.

> ⚠️ Gap: Source mentions **AutoRoute** for establishing routes from an attack box to target networks through a pivot host but omits the tool's specific command-line syntax.

- **Dangerous / misconfigured settings**:
    - **Dual-homed** workstations connected to both enterprise and operational/management networks.
    - Permissive firewall rules allowing inbound traffic on common ports like **80** or **443** for web servers.

## Traffic Tunneling

When to use: Obfuscating **C2** instructions or exfiltrating data by masking traffic within legitimate protocols to avoid detection.

> ⚠️ Gap: Source defines tunneling through **HTTP/HTTPS** (GET/POST) and **SSH over transport protocols** but does not provide specific tool commands for creating these tunnels.

- **Gotchas**: **Source ports** are generated on the client-side; ensure listeners are configured to accept connections from the intended payload ports.

Would you like me to find sources for specific AutoRoute or proxychain commands to fill these gaps?