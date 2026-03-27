### Pivoting, Tunneling, and Lateral Movement Fundamentals

#### Core Concept Comparison

Understanding the distinction between these techniques is critical for selecting the correct approach during an engagement.

|Technique|Goal|Practical Application|
|:--|:--|:--|
|**Lateral Movement**|Further access to hosts, applications, and services within the **same** network environment.|Using compromised local admin credentials to access another Windows host on the same subnet to escalate domain privileges.|
|**Pivoting**|Crossing network boundaries to move **deeper** into isolated or unreachable network segments.|Utilizing a **dual-homed** engineering workstation to bridge the gap between an enterprise network and an operational environment.|
|**Tunneling**|**Encapsulating** network traffic within another protocol to obfuscate actions or bypass security.|Masking Command & Control (C2) traffic inside HTTP GET/POST requests to appear as legitimate web traffic.|

---

#### Step 1: Initial Host Assessment

Upon gaining a foothold, identify if the compromised host can serve as a pivot point to previously unreachable segments.

1. **Check Privilege Levels:** Determine your level of access on the host.
2. **Review Network Connections:** Identify active connections and remote access software like VPNs.
3. **Identify Additional NICs:** Search for multiple Network Interface Controllers (NICs) which indicate the host sits on more than one network.
4. **Document Addressing:** Record all IPv4 addresses and subnet masks to define reachable network portions.

**Interface Identification Commands**

|Operating System|Command|Goal|
|:--|:--|:--|
|**Linux / macOS**|`ifconfig`|View NIC identifiers (eth0, eth1, tun0), IP addresses, and traffic stats.|
|**Windows**|`ipconfig`|View adapters, IPv4/IPv6 addresses, and the **Default Gateway**.|

---

#### Step 2: Routing Table Analysis

Analyze the host's routing decisions to identify which networks it can reach and where it forwards traffic.

1. **Check the Routing Table:** Determine if the host has routes to target networks via specific gateways.
2. **Identify the Default Gateway:** Traffic destined for any network not in the table is sent here.
3. **Evaluate Interface-Learned Routes:** Note routes learned through directly connected interfaces (e.g., eth0, tun0).

**Routing Commands**

|Tool|Command|Decision Point|
|:--|:--|:--|
|**Netstat**|`netstat -r`|Use to view the kernel IP routing table on Linux or Windows.|
|**IP Route**|`ip route`|Alternative Linux command to view destination networks and associated interfaces.|

---

#### Step 3: Operational Execution
SSH Local Port Forwarding￼￼

￼￼Context:￼￼ Use when a specific service (e.g., MySQL) is bound to the ￼￼localhost￼￼ of a compromised host and is inaccessible from the external network. This technique redirects a local port on your attack machine to a remote port on the pivot host via an SSH tunnel.

￼￼Attack Implications:￼￼ Enables the execution of ￼￼remote exploits￼￼ against locally hosted services as if they were running on your own machine.

￼￼￼￼￼Operational Workflow
Select a method to route traffic from the attack machine through the pivot host.

1. **Establish a Tunnel:** Encapsulate traffic into a permitted protocol (like SSH or HTTP) if direct communication is blocked.
2. **Configure AutoRoute:** Use tools like AutoRoute to enable the attack machine to reach internal target networks through the pivot host.
3. **Leverage Permitted Ports:** Identify open ports (e.g., HTTP port 80) permitted through firewalls to gain further network entry.
4. **Service Forwarding:** Forward local or remote services to the attack host to access targets on different segments.

**Operational Parameters**

|Placeholder|Description|
|:--|:--|
|`<ATTACK_IP>`|The IP of your attack machine [Ref].|
|`<PIVOT_IP>`|The IP of the compromised host used to cross network boundaries [Ref].|
|`<TARGET_IP>`|The IP of the internal/victim machine on the unreachable segment [Ref].|
|`<PORT>`|The specific logical port bound to an application (e.g., 80, 443).|

---

#### Key Attack Implications

- **Dual-Homed Hosts:** These are primary targets for pivoting because they possess physical NICs connected to different networks.
- **VPN Interfaces (tun0):** The presence of a tunnel interface indicates an active VPN connection, which is often required to reach specific lab or internal environments.
- **Public vs. Private IPs:** **Public IPs** (e.g., `134.122.100.200`) are reachable via the internet and often found in DMZs, while **Private IPs** are only routable within internal networks.