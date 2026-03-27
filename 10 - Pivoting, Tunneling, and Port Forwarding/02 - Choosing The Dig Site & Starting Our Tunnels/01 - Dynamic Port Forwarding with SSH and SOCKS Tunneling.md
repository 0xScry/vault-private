### **SSH Local Port Forwarding**

**Context:** Use when a specific service (e.g., MySQL) is bound to the **localhost** of a compromised host and is inaccessible from the external network. This technique redirects a local port on your attack machine to a remote port on the pivot host via an SSH tunnel.

**Attack Implications:** Enables the execution of **remote exploits** against locally hosted services as if they were running on your own machine.

#### **Operational Workflow**

1. **Identify Local Services:** Scan the pivot host to identify open SSH ports and potential local-only services.
2. **Establish Tunnel:** Forward a local port to the remote service port.
3. **Verify Connection:** Use local tools to confirm the service is reachable through the forwarded port.

#### **Command Reference**

| Command                                                                                                             | Purpose                                                     |
| :------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------------------- |
| `ssh -L <LOCAL_PORT>:localhost:<REMOTE_PORT> <USERNAME>@<PIVOT_IP>`                                                 | Forwards a single remote port to a local port.              |
| `ssh -L <LOCAL_PORT_1>:localhost:<REMOTE_PORT_1> -L <LOCAL_PORT_2>:localhost:<REMOTE_PORT_2> <USERNAME>@<PIVOT_IP>` | Forwards **multiple ports** simultaneously.                 |
| `netstat -antp \|grep <LOCAL_PORT>`                                                                                 |                                                             |
| `nmap -v -sV -p<LOCAL_PORT> localhost`                                                                              | Queries the forwarded port to identify the service version. |

---

### **Dynamic Port Forwarding (SOCKS Tunneling)**

**Context:** Use when you need to interact with an **entire internal subnet** that is not directly routable from your attack host. This is ideal when you do not yet know which specific services or hosts exist on the target network.

**How it Works:** SSH acts as a **SOCKS server**. A SOCKS client (like Proxychains) on the attack host intercepts tool traffic and routes it through the established SSH tunnel to the internal network.

#### **Operational Workflow**

1. **Enable Dynamic Forwarding:** Start an SSH session with a SOCKS listener.
2. **Configure Proxychains:** Point the proxy tool to the local SOCKS listener.
3. **Route Tools:** Prepend `proxychains` to any networking tool to tunnel its traffic.

#### **Command Reference**

|Command|Purpose|
|:--|:--|
|`ssh -D <LOCAL_SOCKS_PORT> <USERNAME>@<PIVOT_IP>`|Enables dynamic port forwarding on a specified local port.|
|`tail -4 /etc/proxychains.conf`|Verifies the Proxychains configuration.|
|`proxychains nmap -v -sn <TARGET_NET_RANGE>`|Performs a sweep of the internal network through the tunnel.|

---

### **Pivoting via Proxychains**

**Attack Implications:** Hides the IP of the attack host; the target sees the **IP of the pivot host** as the source of the traffic.

#### **Operational Limitations & Failures**

|Condition|Requirement/Impact|
|:--|:--|
|**Nmap Scans**|**Must** use a **full TCP connect scan (`-sT`)**. Proxychains cannot understand partial packets or half-open connections.|
|**Host Discovery**|Traditional ICMP pings often fail. Use **`-Pn`** to disable host-alive checks, especially against Windows targets with active firewalls.|
|**SOCKS Type**|**SOCKS4** (no authentication/UDP). **SOCKS5** (supports authentication and UDP).|

---

### **Internal Enumeration & Access**

Once a tunnel is established, standard tools can be proxied to interact with internal targets.

#### **Metasploit via Proxychains**

To use Metasploit modules against internal targets:

1. Launch the console: `proxychains msfconsole`.
2. All traffic from auxiliary or exploit modules will now route through the SOCKS tunnel.

#### **GUI Access (RDP)**

If an internal host has RDP (Port 3389) open, use `xfreerdp` to gain graphical access.

|Command|Goal|
|:--|:--|
|`proxychains nmap -v -Pn -sT <TARGET_IP>`|Detailed scan of a specific internal host.|
|`proxychains xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>`|Establishes an RDP session through the pivot host.|

**Note:** When using `xfreerdp`, you must accept the RDP certificate before the session initializes.