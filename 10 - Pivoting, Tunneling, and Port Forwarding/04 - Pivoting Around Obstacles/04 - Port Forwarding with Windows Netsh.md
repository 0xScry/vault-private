### **Port Forwarding with Windows Netsh**

#### **Overview**

**Netsh** is a built-in Windows command-line utility used for network configuration. In a penetration testing context, it is used to perform **port forwarding**, allowing an attacker to reach internal resources that are not directly accessible from the attack machine.

**When to Use:**

- When you have compromised a Windows workstation (e.g., via social engineering or phishing) that has access to an **internal network**.
- When you need to **pivot** from a compromised host to a remote internal target.
- When you need to route traffic through a pivot host to access services like **RDP** on a protected internal server.

---

#### **Operational Workflow**

1. **Configure the Port Proxy:** Set up the pivot host to listen on a specific port and forward all received data to the internal target's IP and port.
2. **Verify the Forward:** Confirm the configuration is active and correctly mapped.
3. **Establish Connection:** Connect from the attack machine to the pivot host’s listening port to reach the final destination.

---

#### **Command Reference**

| Parameter        | Description                                                                               |
| :--------------- | :---------------------------------------------------------------------------------------- |
| `listenport`     | The port on the pivot host that will listen for incoming traffic from the attack machine. |
| `listenaddress`  | The IP address of the pivot host where the proxy will listen.                             |
| `connectport`    | The destination port on the internal target machine (e.g., 3389 for RDP).                 |
| `connectaddress` | The internal IP address of the target machine.                                            |

---

#### **Implementation**

**Step 1: Configure Port Forwarding on Pivot Host** Run this command on the compromised Windows host to create the proxy.

```
netsh.exe interface portproxy add v4tov4 listenport=<PORT> listenaddress=<PIVOT_IP> connectport=<PORT> connectaddress=<TARGET_IP>
```

**Step 2: Verify Configuration** Use this to ensure the `v4tov4` (IPv4 to IPv4) mapping is correct.

```
netsh.exe interface portproxy show v4tov4
```

**Step 3: Connect to Target via Pivot** From the attack machine, point your tools (such as `xfreerdp`) at the pivot host's listening port. The traffic will be routed automatically to the internal target.

```
xfreerdp /v:<PIVOT_IP>:<PORT> /u:<USERNAME> /p:<PASSWORD>
```

---

#### **Technical Considerations**

- **Attack Implications:** Successfully configuring a portproxy **unlocks access** to internal services that were previously hidden behind the pivot host's network boundary.